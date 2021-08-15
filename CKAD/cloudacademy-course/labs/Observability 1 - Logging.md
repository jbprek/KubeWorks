````bash
# Create namespace
kubectl create namespace logs
kubectl config set-context --current --namespace=logs
kubectl config view --minify | grep namespace:

#2. Create a multi-container Pod that runs a server and a client that sends requests to the server:
cat << 'EOF' > pod-logs.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: logs
  name: pod-logs
spec:
  containers:
  - name: server
    image: busybox:1.30.1
    ports:
    - containerPort: 8888
    # Listen on port 8888
    command: ["/bin/sh", "-c"]
    # -v for verbose mode
    args: ["nc -p 8888 -v -lke echo Received request"]
    readinessProbe:
      tcpSocket:
        port: 8888
  - name: client
    image: busybox:1.30.1
    # Send requests to server every 5 seconds
    command: ["/bin/sh", "-c"]
    args: ["while true; do sleep 5; nc localhost 8888; done"]
EOF
kubectl create -f pod-logs.yaml
#3. Retrieve the logs (standard output messages) from the server container:

Copy code
1
kubectl logs pod-logs server

#4. Display the most recent log (--tail=1) including the timestamp and stream (-f for follow) the logs from the client container:

kubectl logs -f --tail=1 --timestamps pod-logs client

#5. Press ctrl+c to stop streaming the logs.

#6. Create an Apache web server and allow access to it via a load balancer:

cat << 'EOF' > pod-webserver.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: logs
  name: webserver-logs
spec:
  containers:
  - name: server
    image: httpd:2.4.38-alpine
    ports:
    - containerPort: 80
    readinessProbe:
      httpGet:
        path: /
        port: 80
EOF
kubectl create -f pod-webserver.yaml
#The expose command uses the container port for the Service's port when the --port option isn't provided. In this case, the Service uses port 80 (HTTP).
kubectl expose pod webserver-logs --type=LoadBalancer

#7. Watch the output of get services until the EXTERNAL-IP column has a DNS address:
watch kubectl get services

#8. Copy the DNS address in the External-IP column and press ctrl+c to quit watching the output.

#9. Navigate to the DNS address in a new browser tab to confirm the Service has exposed the Pod over the Internet: 

# 10 Send some HTTP 200 404

#11. Display the logs for the webserver Pod:
kubectl logs webserver-logs

#12. Retrieve the last 10 lines from the conf/httpd.conf file:
kubectl exec webserver-logs -- tail -10 conf/httpd.conf

#13. Copy the conf/httpd.conf from the container to the bastion host:
kubectl cp webserver-logs:conf/httpd.conf local-copy-of-httpd.conf


````
#Kubernetes Logging Using a Logging Agent and the Sidecar Pattern

##Introduction
The sidecar multi-container pattern uses a "sidecar" container to extend the primary container in the Pod. In the context of logging, the sidecar is a logging agent. The logging agent streams logs from the primary container, such as a web server, to a central location that aggregates logs. To allow the sidecar access to the log files, both containers mount a volume at the path of the log files. In this lab step, you will use an S3 bucket to collect logs. You will use a sidecar that uses Fluentd, a popular data collector often used as a logging layer, with an S3 plugin installed to stream log files in the primary container to S3.
````bash
#1. Store the name of the logs S3 bucket created for you by the Cloud Academy Lab environment:
s3_bucket=$(aws s3api list-buckets --query "Buckets[].Name" --output table | grep logs | tr -d \|)
#2. To verify that the Amazon S3 bucket name was retrieved successfully, enter the following:
echo $s3_bucket
#3. Create a ConfigMap that stores the fluentd configuration file:
cat << EOF > fluentd-sidecar-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluent.conf: |
    # First log source (tailing a file at /var/log/1.log)
    <source>
      @type tail
      format none
      path /var/log/1.log
      pos_file /var/log/1.log.pos
      tag count.format1
    </source>

    # Second log source (tailing a file at /var/log/2.log)
    <source>
      @type tail
      format none
      path /var/log/2.log
      pos_file /var/log/2.log.pos
      tag count.format2
    </source>

    # S3 output configuration (Store files every minute in the bucket's logs/ folder)
    <match **>
      @type s3

      s3_bucket $s3_bucket
      s3_region us-west-2
      path logs/
      buffer_path /var/log/
      store_as text
      time_slice_format %Y%m%d%H%M
      time_slice_wait 1m
      
      <instance_profile_credentials>
      </instance_profile_credentials>
    </match>
EOF
kubectl create -f fluentd-sidecar-config.yaml
# ConfigMaps allow you to separate configuration from the container images. This increases the reusability of images. 
# ConfigMaps can be mounted into containers using Kubernetes Volumes. 
# Explaining the fluent.conf configuration file format is outside of the scope of this Lab. 
# Just know that two log sources are configured in the /var/log directory and their log messages will be tagged with count.format1 and count.format2. The primary container in the Pod will stream logs to those two files. 
# he configuration also describes streaming all the logs to the S3 logs bucket in the match section.  

#4. Create a multi-container Pod using a fluentd logging agent sidecar (count-agent): 
cat << 'EOF' > pod-counter.yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    command: ["/bin/sh", "-c"]
    args:
    - >
      i=0;
      while true;
      do
        # Write two log files along with the date and a counter
        # every second
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    # Mount the log directory /var/log using a volume
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-agent
    image: lrakai/fluentd-s3:latest
    env:
    - name: FLUENTD_ARGS
      value: -c /fluentd/etc/fluent.conf
    # Mount the log directory /var/log using a volume
    # and the config file
    volumeMounts:
    - name: varlog
      mountPath: /var/log
    - name: config-volume
      mountPath: /fluentd/etc
  # Use host network to allow sidecar access to IAM instance profile credentials
  hostNetwork: true
  # Declare volumes for log directory and ConfigMap
  volumes:
  - name: varlog
    emptyDir: {}
  - name: config-volume
    configMap:
      name: fluentd-config
EOF
kubectl create -f pod-counter.yaml
#The count container writes the date and a counter variable ($i) in two different log formats to two different log files in the /var/log directory
# every second. The /var/log directory is mounted as a Volume in both the primary count container and the count-agent sidecar 
# so both containers can access the logs. The sidecar also mounts the ConfigMap to access the fluentd configuration file. 
# By using a ConfigMap, the same sidecar container can be used for any configuration compared to storing the configuration in the image 
# and having to manage separate container images for each configuration.

#5. In order to view the logs in S3, open the AWS Management Console by clicking Open Environment in the upper-left section of this lab: 
````
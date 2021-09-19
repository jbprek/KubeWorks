#Check 1: Create and Consume a Secret using a Volume
Create a secret named app-secret in the yqe Namespace that stores key-value pair of password=abnaoieb2073xsj

Create a Pod that consumes the app-secret Secret using a Volume that mounts the Secret in the /etc/app directory. The Pod should be named app and run a memcached container.

#Check 2: Update Deployment with New Service Account
A Deployment named secapp has been created in the app namespace and currently uses the default ServiceAccount. Create a new ServiceAccount named secure-svc in the app namespace and then use it within the existing secapp Deployment, ensuring that the replicas now run with it.


#Check 3: Pod Security Context Configuration
Create a pod named secpod in the dnn namespace which includes 2 containers named c1 and c2. Both containers must be configured to run the bash image, and should execute the command /usr/local/bin/bash -c sleep 3600. Container c1 should run as user ID 1000, and container c2 should run as user ID 2000. Both containers should use file system group ID 3000.


#Check 4: Pod Resource Constraints
Create a new Pod named web1 in the ca100 namespace using the nginx image. Ensure that it has the following 2 labels env=prod and type=processor. Configure it with a memory request of 100Mi and a memory limit at 200Mi. Expose the pod on port 80.


#Check 5: Create a Pod with Config Map Environment Vars
Create a new ConfigMap named config1 in the ca200 namespace. The new ConfigMap should be created with the following 2 key/value pairs:

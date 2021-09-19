#Check 1: Create Nginx Pod with Liveness HTTP Get Probe (SOS)
Create a new Pod named nginx in the ca1 namespace using the nginx image. Ensure that the pod listens on port 80 and configure it with a Liveness HTTP GET probe with the following configuration:

#Check 1: Solution
Probe Type: httpGet
Path: /
Port: 80
Initial delay seconds: 10
Polling period seconds: 5


#Check 2: Hosting Service Not Working
A Service in the hosting Namespace is not responding to requests. Determine which Service is not working and resolve the underlying issue so the Service begins responding to requests.


#Check 3: Pod Log Analysis
The ca2 namespace contains a set of pods. Pods labelled with app=test, or app=prod have been designed to log out a static list of numbers. Run a command that combines the pod logs for all pods that have the label app=prod and then get a total row count for the combined logs and save this result out to the file /home/ubuntu/combined-row-count-prod.txt.

##Solution commands
kubectl logs -n ca2 -l app=prod | wc -l > /home/ubuntu/combined-row-count-prod.txt

#Check 4: Pod Diagnostics
A pod named skynet has been deployed into the ca2 namespace. This pod has the following file /skynet/t2-specs.txt located within it, containing important information. You need to extract this file and save it to the following location /home/ubuntu/t2-specs.txt.

##Notes
Can be done with 'kubectl cp'
### Alternative Solution commands
kubectl exec -n ca2 skynet -- cat /skynet/t2-specs.txt > /home/ubuntu/t2-specs.txt

#Check 5: Pod CPU Utilization
Find the Pod that has the highest CPU utilization in the matrix namespace. Write the name of this pod into the following file: /home/ubuntu/max-cpu-podname.txt

##Solution commands
kubectl top pods -n matrix --sort-by=cpu --no-headers=true | head -n1 | cut -d" " -f1 > /home/ubuntu/max-cpu-podname.txt

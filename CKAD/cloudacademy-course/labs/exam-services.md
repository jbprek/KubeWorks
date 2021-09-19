Check 1: Create and Configure a Basic Pod
Create a Pod in the red Namespace with the following configuration:

The Pod is named basic
The Pod uses the nginx:stable-alpine-perl image for its only container
Restart the Pod only OnFailure
Ensure port 80 is open to TCP traffic

Check 2: Expose Pod
Expose the Pod named basic in the red Namespace, ensuring it has the following settings:

Service name is cloudacademy-svc
Service port is 8080
Target port is 80
Service type is ClusterIP

Check 3: Expose existing Deployment
A Deployment named cloudforce has been created in the ca1 Namespace. You must now expose this Deployment as a NodePort based Service using the following settings:

Service name is cloudforce-svc
Service type is NodePort
Service port is 80
NodePort is 32080



Check 4: Fix Networking Issue
A Deployment named t2 has been created in the skynet Namespace, and has been exposed as a ClusterIP based Service named t2-svc. The t2-svc Service when contacted should return a valid HTTP response, but unfortunately, this is currently not yet the case. Please investigate and fix the problem. When you have applied the fix, run the following command to save the HTTP response:

kubectl run client -n skynet --image=appropriate/curl -it --rm --restart=Never -- curl http://t2-svc:8080 > /home/ubuntu/svc-output.txt

Check 5: Secure Pod Networking
The following two Pods have been created in the sec1 Namespace:

pod1 has the label app=test
pod2 has the label app=client
A NetworkPolicy named netpol1 has also been established in the sec1 Namespace but is currently blocking traffic sent from pod2 to pod1. Update the NetworkPolicy to ensure that pod2 can send traffic to pod1.


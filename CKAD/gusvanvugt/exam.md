It's time for the sample exam. I have created a couple of nice assignments for you. Make sure that you can succeed all of them within a two hour timeframe. The two hours is a part of the game. If you can't do it within two hours, you need to increase on your speed before taking the actual exam. So I'm first going to read all the assignments for you. Feel free to pause and work on the assignment and then continue. That's what I would advise you to do. And once you're done, there are solutions for all of the assignments in the next couple of videos. So watch the solutions to verify that you are more or less in the right direction. So the assignments, here we go.

1. To start with first assignment is about working with NameSpaces. So create the NameSpace ckad-ns1 in your cluster. In this NameSpace, run the following Pods: a Pod with the name pod-a, running httpd server image and a Pod with the name pod-b running the Nginx server image as well as the alpine image.

2. Second assignment is finding Pods. Use the appropriate command to find all Pods that have the label as "app" set to "nginx" sorted by name in alphabetical order using kubectl options. So no Linux grep but kubectl options.

3. Next is ConfigMaps. In the ckad-ns2 NameSpace, run a Pod with the name alpine-pod. This Pod should run a container based on the alpine image. And in its container, two variables must be set, using ConfigMaps. First localport is set to localhost:8082, and then external url is set to linux.com.

4. Next assignment is about Sidecars. Create a Multi-container Pod with the name sidecar-pod, that runs in the ckad-ns3 namespace. The primary container is running busybox, and writes the output of the data command to the var/log date.log file every five seconds. The second container should run as a sidecar and provide Nginx web- access to this file, using a hostPath shared volume. Mount on usr/share/nginx/html. Next, make sure that the image for this container is only pulled if it's not available on the local system yet.

5. The next assignment is inspecting containers. The Pod my-server is running three containers, file-server, log-server, db server. And when starting it, the log-server fails. So you'll see appropriate command to analyze what is going wrong. 

6. Next is using Probes. Create a Pod that ruins the httpd webserver. The webserver should be offering its services on port 80 and running the ckad-ns3 namespace. This Pod should use a readiness probe that gets 60 seconds to complete and probe should check the availability of webserver document documentary root. Which is usr/local/apache2/htdocs in the default httpd image before the start but also during operation. 

7. Then creating a deployment. Write a manifest file with the name nginx-exam.yaml that meets the following requirements. It starts five replicas that run the nginx:1.18 image. Each Pod has the label app is webshop and create the deployment such that while updating the existing Pods are terminated before new Pods are created to replace them. And the deployment itself should use the label service is nginx. 

8. Next is exposing applications. In the ckad-ns6 NameSpace, create a deployment that runs the nginx1.19 image and give it the name nginx-deployment and ensure it runs three replicas. And after verifying that the deployment run successfully, expose it such that users that are external to the cluster can reach it by addressing the Node Port 32000 on the Kubernetes cluster node. 

9. Next assignment is about using network policies. Create a YAML file with the name my-nw-policy that runs two Pods and a NetworkPolicy. The first Pod should run an Nginx server with the default settings. The second Pod should run a Busybox image with sleep at 3600 command. Use a NetworkPolicy to restrict traffic between Pods in the following way. Access to the Nginx server is allowed for the BusyBox Pod and the BusyBox Pod is not restricted in any way. 

10. Next is about using storage. All the objects in this assignment should be created in the ckad-1311 namespace. Create a Persistent Volume with the name 1311-pv. It should provide two gigabytes of storage and read-write access to multiple clients simultaneously. Use any storage type you like. 
11. Next, create a Persistent Volume Claim that requests one gigabyte from any persistent volume that allows multiple clients simultaneous read-write access. And the name of that object should be 1311-pvc. And finally, create a Pod with the name 1311-pod that uses this Persistent Volume. It should run an Nginx image and amount of volume on the directory/webdata. 
12. Then we are going to use Helm. Use the Helm package manager to learn the bitnami Nginx application according to the following requirements. The application should start three Pods and the name of the application is set to helmginx. Following is a quota. Run a Deployment with the name restrictginx, with three Pods where every Pod initially requests 64 megabytes of Ram with an upper limit of 256 megabytes of Ram. 
13. And finally, using a ServiceAccount. Create a Pod with the name allaccess. And also create a ServiceAccount with the name allaccess and ensure that the Pod is using the ServiceAccount where you don't have to set up any additional role based access control. That's all. Remember you have two hours, good luck. And don't forget to watch the solutions once you are done.

table of contents
Settings
queue


#Check 1: Create and Manage Deployments
Create a deployment named webapp in the zap namespace. Use the nginx:1.17.8 image and set the number of replicas initially to 2. Next, scale the current deployment up from 2 to 4. Finally, update the deployment to use the newer nginx:1.19.0 image.


#Check 2: Create Pod Labels
Add an additional label app=cloudacademy to all pods currently running in the gzz namespace that have the label env=prod


#Check 3: Rollback Deployment
The nginx container running within the cloudforce deployment in the fre namespace needs to be updated to use the nginx:1.19.0-perl image. Perform this deployment update and ensure that the command used to perform it is recorded in the tracked rollout history.
**NOTE** --record option

#Check 4: Configure Pod AutoScaling
A deployment named eclipse has been created in the xx1 namespace. This deployment currently consists of 2 replicas. Configure this deployment to autoscale based on CPU utilisation. The autoscaling should be set for a minimum of 2, maximum of 4, and CPU usage of 65%.


#Check 5: Create CronJob
Create a cronjob named matrix in the saas namespace. Use the radial/busyboxplus:curl image and set the schedule to */10 * * * *. The job should run the following command: curl www.google.com


#Check 6: Filter and Sort Pods
Get a list of all pod names running in the rep namespace which have their colour label set to either orange, red, or yellow. The returned pod name list should contain only the pod names and nothing else. The pods names should be ordered by the cluster IP address assigned to each pod. The resulting pod name list should be saved out to the file /home/ubuntu/pod001

The following list is an example of the required output:

pod6
pod17
pod3
pod16
pod15
pod13


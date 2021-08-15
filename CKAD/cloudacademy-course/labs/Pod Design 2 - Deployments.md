````bash

# 1 Create namespace
kubectl create namespace deployment
# Set namespace as the default for the current context
kubectl config set-context --current --namespace=deployment

kubectl config view --minify | grep namespace:

# 2. Use the kubectl create command to generate a Deployment manifest for you:
kubectl create deployment --image=httpd:2.4.38 web-server --dry-run=client -o yaml

#3. Run the previous command without the dry run to actually create the Deployment:
kubectl create deployment --image=httpd:2.4.38 web-server

#4. Describe the Deployment to get details about the Deployment and its Pods:
kubectl describe deployments web-server


#5. Confirm that there is one Pod running, which matches the desired state for the number of replicas:
kubectl get pods

#6. Scale up the number of replicas in the Deployment to six (6):
kubectl scale deployment web-server --replicas=6

#7. Confirm that there are now six (6) Pods running:
kubectl get pods

#8. View the Deployment's rollout history to confirm only one revision exists:
kubectl rollout history deployment web-server

# 9. Use the edit command to open an editor to edit the Deployment:
kubectl edit deployment web-server --record

#10. Perform the following actions to update the deployment so that the Pods' containers' open port 80:
Add under 'spec.template.spec.containers.image' child ports.containerPort value with value 80

#11. Press escape (this will remove the ---INSERT--- message at the bottom of the window) and type :wq followed by enter to save (write, quit) the file.
# A new desired state is created for the Pods. This in turn kicks off a rolling update for the Pods.

#12. Confirm the rollout was successful:
kubectl rollout status deployment web-server

#13 13. View the rollout history and observe the CHANGE-CAUSE column displays the command you issued to edit the Deployment:
kubectl rollout history deployment web-server
#14. Set the Pods' container image to httpd:2.4.38-alpine: 
kubectl set image deployment web-server httpd=httpd:2.4.38-alpine --record

#15. View the latest events for the Deployment:
kubectl describe deployments web-server

#16. Rollback the previous change:
kubectl rollout undo deployment web-server

#17. Expose the webserver Deployment to the internet by creating a Service of type LoadBalancer:
kubectl expose deployment web-server --type=LoadBalancer --port=80

#18. Watch the output of get services until the EXTERNAL-IP column has a DNS address:
watch kubectl get services

#19. Copy the DNS address in the External-IP column and press ctrl+c to quit watching the output.

 
#20. Navigate to the DNS address in a new browser tab to confirm the Service has exposed the Deployment over the Internet: 
````
- [Link with solutions](https://github.com/dgkanatsios/CKAD-exercises/blob/master/a.core_concepts.md)
1. Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace
2. Create the pod that was just described using YAML
3. Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output
   ~~k run --image busybox -restart=Never --command --env~~ 
   k run busybox --image busybox --restart=Never -rm -it --command -- env
4. Create a busybox pod (using YAML) that runs the command "env". Run it and see the output
5. Get the YAML for a new namespace called 'myns' without creating it
6. Get the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it
7. Get pods on all namespaces
10. Create a pod with image nginx called nginx and expose traffic on port 80
11. Change pod's image to nginx:1.7.1. Observe that the container will be restarted as soon as the image gets pulled
12. Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'
13. Get pod's YAML
14. Get information about the pod, including details about potential issues (e.g. pod hasn't started)
15. Get pod logs
16. If pod crashed and restarted, get logs about the previous instance
17. Execute a simple shell on the nginx pod
18. Create a busybox pod that echoes 'hello world' and then exits
19. Do the same, but have the pod deleted automatically when it's completed
20. Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod
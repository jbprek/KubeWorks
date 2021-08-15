# Declaring Labels

- ? Are applied only to pods
## Using run command
````
$ kubectl run labeled-pod --image=nginx --restart=Never --labels=tier=backend,env=dev
  ````
## declaratively 
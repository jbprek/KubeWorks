# Secrets
## Creating a Secret Imperative way
command: `kubectl create secret`
has the following options
- generic (Most commonly used, has the same cmdline option as kubectl create configmap)
- docker-registry 
- tls
  
In most cases, you will likely deal with the type generic, which provides the same command-line options to point to the source of the configuration data as kubectl create configmap:
- Literal values, which are key-value pairs as plain text.
- A file that contains key-value pairs and expects them to be environment variables.
- A file with arbitrary contents.
- A directory with one or many files.

### Examples
1. Literal values

````
$ kubectl create secret generic db-creds --from-literal=pwd=s3cre!
 secret/db-creds created
````

2. File containing environment variables`
````
$ kubectl create secret generic db-creds --from-env-file=secret.env
secret/db-creds created
````
3. SSH key file
````
$ kubectl create secret generic ssh-key --from-file=id_rsa=~/.ssh/id_rsa
secret/db-creds created
````

## Creating a Secret Declarative way

1. base64 encode secret value
````
$ echo -n 's3cre!' | base64
czNjcmUh
````

2. create deployment descriptor
````
vi db-creds.yaml
````

````
apiVersion: v1
kind: Secret
metadata:
  name: db-creds
type: Opaque
data:
  password: czNjcmUh
````

3. create secret
````
kubectl apply -f db-creds.yaml
````
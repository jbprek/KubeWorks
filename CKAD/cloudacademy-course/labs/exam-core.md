##Check 1: Create and Configure a Basic Pod
Create a Pod in the cre Namespace with the following configuration:

The Pod is named basic
The Pod uses the nginx:stable-alpine-perl image for its only container
Restart the Pod only OnFailure
Ensure port 80 is open to TCP traffic

##Check 2: Create a Namespace and Launch a Pod within it with Labels
Create a new Namespace named workers and within it launch a Pod with the following configuration:

The Pod is named worker
The Pod uses the busybox image for its only container
Restart the Pod Never
Command: /bin/sh -c "echo working... && sleep 3600"
Label 1: company=acme
Label 2: speed=fast
Label 3: type=async


##Check 3: Update the Label on a Running Pod
The ca200 namespace contains a running Pod named compiler. Without restarting the pod, update and change it's language label from java to python.

##Check 4: Get the Pod IP Address using JSONPath  (TODO)
Discover the Pod IP address assigned to the pod named ip-podzoid running in the ca300 namespace using JSONPath (hint: use -o jsonpath). Once you've established the correct kubectl command to do this, save the command (not the result) into a file located here: /home/ubuntu/podip.sh

##Check 5: Generate Pod YAML Manifest File
Generate a new Pod manifest file which contains the following configuration:

Pod name: borg1
Namespace to launch in: core-system
Container image: busybox
Command: /bin/sh -c "echo borg.running... && sleep 3600"
Restart policy: Always
Pod label: platform=prod
Environment variable: system=borg
Save the resulting manifest to the following location: /home/ubuntu/pod.yaml


##Check 6: Launch Pod and Configure it's Termination Shutdown Time (TODO)
Launch a new web server Pod in the sys2 namespace with the following configuration:

Pod name: web-zeroshutdown
Container image: nginx
Restart policy: Never
Ensure the pod is configured to terminate immediately when requested to do so by configuring it's terminationGracePeriodSeconds setting.

#TODO

- jsonpath
- terminationGracePeriodSeconds
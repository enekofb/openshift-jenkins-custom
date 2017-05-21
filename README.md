# openshift-jenkins-custom
Using openshift source-to-image to create a proper Jenkis image.

#Getting started

- Following the instructions found here (https://docs.openshift.com/container-platform/3.4/using_images/other_images/jenkins.html#jenkins-as-s2i-builder)

- Creat an image stream for the new image set with `oc create -f jenkins-imagestream.yaml`

- Created build config `oc create -f jenkins-buildconfig.yaml`

- Started build `oc start-build custom-jenkins-build`

Notice that is creating the new images (downloading base image,etc) and it could take time so do wait for a while until finished.
```
oc logs custom-jenkins-build-2-build

Cloning "https://github.com/enekofb/openshift-jenkins-custom.git" ...
	Commit:	364ca620ff8e60a1b42b76bf51dfd888d4a3bf82 (initial commit)
	Author:	Eneko Fernandez <eneko.fernandez@opencredo.com>
	Date:	Sat May 20 23:15:01 2017 +0100
Enekos-MacBook-Pro:openshift-jenkins-custom eneko$ oc logs custom-jenkins-build-2-build
Cloning "https://github.com/enekofb/openshift-jenkins-custom.git" ...
	Commit:	364ca620ff8e60a1b42b76bf51dfd888d4a3bf82 (initial commit)
	Author:	Eneko Fernandez <eneko.fernandez@opencredo.com>
	Date:	Sat May 20 23:15:01 2017 +0100
---> Copying repository files ...
---> Installing 1 Jenkins plugins from plugins/ directory ...

Pushing image 172.30.1.1:5000/myproject/custom-jenkins:1 ...
Pushed 0/7 layers, 19% complete
Pushed 1/7 layers, 15% complete
Pushed 2/7 layers, 29% complete
Pushed 3/7 layers, 43% complete
Pushed 4/7 layers, 63% complete
Pushed 4/7 layers, 85% complete
Pushed 5/7 layers, 90% complete
Pushed 6/7 layers, 95% complete
Pushed 7/7 layers, 100% complete
Push successful
```
- Created tags `oc get istag`

```
NAME               DOCKER REF                                                                                                         UPDATED         IMAGENAME
custom-jenkins:1   172.30.1.1:5000/myproject/custom-jenkins@sha256:e68b55c535a44a2b14456b6dfbc336dc3a7b47c174607b431df7ee230255e7a5   3 minutes ago   sha256:e68b55c535a44a2b14456b6dfbc336dc3a7b47c174607b431df7ee230255e7a5
```

- Created image-stream `oc get is` 
```
NAME             DOCKER REPO                                TAGS      UPDATED
custom-jenkins      1         4 minutes ago

```

- Create template for new image `oc create -f jenkins-template.yaml`

````
Enekos-MacBook-Pro:openshift-jenkins-custom eneko$ oc get template custom-jenkins-ephimeral -n openshift
NAME                       DESCRIPTION                                       PARAMETERS    OBJECTS
custom-jenkins-ephimeral   Jenkins service, without persistent storage....   6 (all set)   6
````

- Create application for new image `oc new-app custom-jenkins-ephimeral`

```
--> Deploying template "openshift/custom-jenkins-ephimeral" to project myproject

     Jenkins (Ephemeral)
     ---------
     Jenkins service, without persistent storage.
     WARNING: Any data stored will be lost upon pod destruction. Only use this template for testing.

     A Jenkins service has been created in your project.  Log into Jenkins with your OpenShift account.  The tutorial at https://github.com/openshift/origin/blob/master/examples/jenkins/README.md contains more information about using this template.


     * With parameters:
        * Jenkins Service Name=jenkins
        * Jenkins JNLP Service Name=jenkins-jnlp
        * Enable OAuth in Jenkins=true
        * Memory Limit=512Mi
        * Jenkins ImageStream Namespace=openshift
        * Jenkins ImageStreamTag=jenkins:latest

--> Creating resources ...
    route "jenkins" created
    deploymentconfig "jenkins" created
    serviceaccount "jenkins" created
    rolebinding "jenkins_edit" created
    service "jenkins-jnlp" created
    service "jenkins" created
--> Success
    Run 'oc status' to view your app.

```
- Wait until jenkis is running

^CEnekos-MacBook-Pro:openshift-jenkins-custom eneko$ oc get pods
NAME                           READY     STATUS      RESTARTS   AGE
custom-jenkins-build-1-build   0/1       Completed   0          14m
jenkins-1-f4qf8                1/1       Running     0          27s

- And the route has been created 

Enekos-MacBook-Pro:openshift-jenkins-custom eneko$ oc get route
NAME      HOST/PORT                                 PATH      SERVICES   PORT      TERMINATION     WILDCARD
jenkins   jenkins-myproject.192.168.99.101.nip.io             jenkins    <all>     edge/Redirect   None

- Access to its route in the browser, a see what plugins has been installed


- How to add a new plugin

- add it to plugins.txt

- create new build

- delete dc

- start new app

Enekos-MacBook-Pro:openshift-jenkins-custom eneko$ oc delete dc jenkins
deploymentconfig "jenkins" deleted
Enekos-MacBook-Pro:openshift-jenkins-custom eneko$ oc new-app custom-jenkins-ephimeral
--> Deploying template "openshift/custom-jenkins-ephimeral" to project myproject

     Jenkins (Ephemeral)
     ---------
     Jenkins service, without persistent storage.
     WARNING: Any data stored will be lost upon pod destruction. Only use this template for testing.

     A Jenkins service has been created in your project.  Log into Jenkins with your OpenShift account.  The tutorial at https://github.com/openshift/origin/blob/master/examples/jenkins/README.md contains more information about using this template.


     * With parameters:
        * Jenkins Service Name=jenkins
        * Jenkins JNLP Service Name=jenkins-jnlp
        * Enable OAuth in Jenkins=true
        * Memory Limit=512Mi
        * Jenkins ImageStream Namespace=myproject
        * Jenkins ImageStreamTag=custom-jenkins:latest

--> Creating resources ...
    error: routes "jenkins" already exists
    deploymentconfig "jenkins" created
    error: serviceaccounts "jenkins" already exists
    error: rolebinding "jenkins_edit" already exists
    error: services "jenkins-jnlp" already exists
    error: services "jenkins" already exists
--> Failed


A question?

- Do we need to recreate the app everytime we change the application?

No, if we have a look in deploymentconfig we have some triggers

```
    triggers:
    - imageChangeParams:
        automatic: true
        containerNames:
        - jenkins
        from:
          kind: ImageStreamTag
          name: ${JENKINS_IMAGE_STREAM_TAG}
          namespace: ${NAMESPACE}
        lastTriggeredImage: ""
      type: ImageChange
```

where one of it is Image change configuration so it means that deployment config controller is listenting
for changes in the image in order to recreate the deployment.

This can be seen by looking at the deployments done for jenkins

```
Enekos-MacBook-Pro:openshift-jenkins-custom eneko$ oc describe dc jenkins
Name:		jenkins
Namespace:	myproject
Created:	6 minutes ago
Labels:		app=custom-jenkins-ephimeral
		template=custom-jenkins-ephimeral-template
Annotations:	openshift.io/generated-by=OpenShiftNewApp
Latest Version:	2
Selector:	name=jenkins
Replicas:	1
Triggers:	Image(custom-jenkins@latest, auto=true), Config
Strategy:	Recreate
Template:
  Labels:		app=custom-jenkins-ephimeral
			name=jenkins
  Annotations:		openshift.io/generated-by=OpenShiftNewApp
  Service Account:	jenkins
  Containers:
   jenkins:
    Image:	172.30.1.1:5000/myproject/custom-jenkins@sha256:31800df7380f6dd8afd3614d7de78539ef79effcf2f8064a3f657757a1bb53e4
    Port:
    Limits:
      memory:	512Mi
    Liveness:	http-get http://:8080/login delay=420s timeout=3s period=10s #success=1 #failure=30
    Readiness:	http-get http://:8080/login delay=3s timeout=3s period=10s #success=1 #failure=3
    Volume Mounts:
      /var/lib/jenkins from jenkins-data (rw)
    Environment Variables:
      OPENSHIFT_ENABLE_OAUTH:		true
      OPENSHIFT_ENABLE_REDIRECT_PROMPT:	true
      KUBERNETES_MASTER:		https://kubernetes.default:443
      KUBERNETES_TRUST_CERTIFICATES:	true
      JNLP_SERVICE_NAME:		jenkins-jnlp
  Volumes:
   jenkins-data:
    Type:	EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:

Deployment #2 (latest):
	Name:		jenkins-2
	Created:	2 minutes ago
	Status:		Complete
	Replicas:	1 current / 1 desired
	Selector:	deployment=jenkins-2,deploymentconfig=jenkins,name=jenkins
	Labels:		app=custom-jenkins-ephimeral,openshift.io/deployment-config.name=jenkins,template=custom-jenkins-ephimeral-template
	Pods Status:	1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Deployment #1:
	Created:	6 minutes ago
	Status:		Complete
	Replicas:	0 current / 0 desired

Events:
  FirstSeen	LastSeen	Count	From				SubObjectPath	Type		Reason			Message
  ---------	--------	-----	----				-------------	--------	------			-------
  6m		6m		1	{deploymentconfig-controller }			Normal		DeploymentCreated	Created new replication controller "jenkins-1" for version 1
  2m		2m		1	{deploymentconfig-controller }			Normal		DeploymentCreated	Created new replication controller "jenkins-2" for version 2
```

where the latests one points to ournew version

```
Deployment #2 (latest):
	Name:		jenkins-2
	Created:	2 minutes ago
	Status:		Complete
	Replicas:	1 current / 1 desired
	Selector:	deployment=jenkins-2,deploymentconfig=jenkins,name=jenkins
	Labels:		app=custom-jenkins-ephimeral,openshift.io/deployment-config.name=jenkins,template=custom-jenkins-ephimeral-template
	Pods Status:	1 Running / 0 Waiting / 0 Succeeded / 0 Failed

```

And we can confirm that the plugin has been installed img maven-metadata-plugin-installed
# openshift-jenkins-custom
Using openshift source-to-image to create a proper Jenkis image.

#Getting started

- Following the instructions found here (https://docs.openshift.com/container-platform/3.4/using_images/other_images/jenkins.html#jenkins-as-s2i-builder)

- Creat an image stream for the new image set with `oc create -f jenkins-imagestream.yaml`

- Created build config `oc create -f jenkins-buildconfig.yaml`

- Started build `oc start-build custom-jenkins-build`

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
- Created tags

```
oc get istag
NAME               DOCKER REF                                                                                                         UPDATED         IMAGENAME
custom-jenkins:1   172.30.1.1:5000/myproject/custom-jenkins@sha256:e68b55c535a44a2b14456b6dfbc336dc3a7b47c174607b431df7ee230255e7a5   3 minutes ago   sha256:e68b55c535a44a2b14456b6dfbc336dc3a7b47c174607b431df7ee230255e7a5
```

- Created image-stream 
```
oc get is
NAME             DOCKER REPO                                TAGS      UPDATED
custom-jenkins      1         4 minutes ago

```

- Create template for new image `oc create -f jenkins-template.yaml`

````
Enekos-MacBook-Pro:openshift-jenkins-custom eneko$ oc get template custom-jenkins-ephimeral -n openshift
NAME                       DESCRIPTION                                       PARAMETERS    OBJECTS
custom-jenkins-ephimeral   Jenkins service, without persistent storage....   6 (all set)   6
````

- Create application for new image

```
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
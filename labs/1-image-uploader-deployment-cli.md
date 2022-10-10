# Deploy a Project From GitHub using the Command Line
This lab you will find your way around the ``oc`` CLI by deploying a PhP project from GitHub. This app will upload images and to the webpage and show the uploaded images in a gallery.

## 1. Create a New Project
- Open a Linux shell and make sure you are login to the ``oc`` cli by clicking on your username in the top right of the web console, then copying the login command and pasting it into your command line. 
  - This presumes you have the ``oc`` tool already installed. If not, install from https://docs.openshift.com/container-platform/4.10/cli_reference/openshift_cli/getting-started-cli.html. Remember if you are using Git Bash on Windows you need to install the *windows* executable.
- Create a new project (replacing XX with your login number):
```
       $ oc new-project oseXX-iu-cli --display-name="My Image Uploader"
```
- You should see output something like:
```
        Now using project "ose4-iu-cli" on server "https://api.ose.openshift.conygre.com:6443"
```

## 2. Add the App to your project
- To create a new PhP application enter the following command:
```
        $ oc new-app php~https://github.com/OpenShiftInAction/image-uploader.git --name image-uploader --labels app=image-uploader-app
```
- ``oc new-app`` will create the new application in the current project. The ``php~`` prefix to the git URL tells OpenShift that you want to build from source (you can specify this with ``--strategy=source`` as well) using a PhP Build Image. With ``--name`` we specify the name of the app, and ``--labels`` (or ``-l``) follows with a comma-separated list of ``labelname=labelvalue`` pairs. 
- Expected output:
```
        --> Found image f2b8dfb (9 days old) in image stream "openshift/php" under tag "7.4-ubi8" for "php"

            Apache 2.4 with PHP 7.4
            -----------------------
            PHP 7.4 available as container is a base platform for building and running various PHP 7.4 applications and frameworks. PHP is an HTML-embedded scripting language. PHP attempts to make it easy for developers to write dynamically generated web pages. PHP also offers built-in database integration for several commercial and non-commercial database management systems, so writing a database-enabled webpage with PHP is fairly simple. The most common use of PHP coding is probably as a replacement for CGI scripts.

            Tags: builder, php, php74, php-74

            * A source build using source code from https://github.com/OpenShiftInAction/image-uploader.git will be created
            * The resulting image will be pushed to image stream tag "image-uploader:latest"
            * Use 'oc start-build' to trigger a new build

        --> Creating resources with label app=image-uploader-app ...
            imagestream.image.openshift.io "image-uploader" created
            buildconfig.build.openshift.io "image-uploader" created
            deployment.apps "image-uploader" created
            service "image-uploader" created
        --> Success
            Build scheduled, use 'oc logs -f buildconfig/image-uploader' to track its progress.
            Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
            'oc expose service/image-uploader'
            Run 'oc status' to view your app.
```
- To create a route to expose your application enter the following command:

```
        $ oc create route edge image-uploader --service=image-uploader
```
- You should see output like:
```
        route.route.openshift.io/image-uploader created
```
- The route URL can be obtained by: ``$ oc get route``. 
- You can open it in a web browser (Chrome in this case) with 
```
        start chrome https://$(oc get route image-uploader --template='{{ .spec.host}}')
```
- Check in the Web Console to see that your app has deployed as expected. Make sure you choose the right project in the Developer perspective!

## 3. Check your build progress
- To check in on the status of your build, run the command
```
        $ oc get builds
```
- Sample output
```
        NAME               TYPE     FROM          STATUS     STARTED          DURATION
        image-uploader-1   Source   Git@549c356   Complete   15 minutes ago   43s     
```
- In this case the build status is ``Complete``. You can get more details on this specific build using ``$ oc describe build image-uploader-1``. 
  
## 4. Scale your app up and down
-  The app can be scaled as follows:
```
        $ oc scale --current-replicas=1 --replicas=2 deployment/image-uploader
```
- Sample output:
```
        deployment.apps/image-uploader scaled
```
- To check it has been scaled as expected, type in ``$ oc get pods``, you should find two running pods prefixed with ``image-uploader``. There may also be a pod that was launched for the build step and is now marked ``Completed``.
- Scale your application back down to 1 pod:
```
        $ oc scale --current-replicas=2 --replicas=1 deployment/image-uploader
```

## 5. Remove your Project (DON'T DO THIS if you are intending to do lab 3!)
Although this project has only one application, it has actually generated a number of resources: a Pod, a Service, a Deployment, a ReplicaSet, a BuildConfig, a Build, and ImageStream and a Route. 
- You can list all these resources using:
```
        oc get all --selector app=image-uploader-app
```
- When you decide you want to remove this application you really want to remove all the other resources associated. If, for example, you just delete the deployment ( ``$ oc delete deployment image-uploader``), the other resources remain.
- Two ways to ensure you remove all the resources:
```
        $ oc delete all --selector app=image-uploader-app # 1. this will clear out all the resources but leave the project in place
        $ oc delete project oseXX-iu-cli #2. this will clear out the project as well
```



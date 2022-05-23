# Deploying a Noted App Using OpenShift Pipelines
In this tutorial we will go through the process of forking a GitHub repository, then deploying a backend from that fork using the OpenShift Pipelines Operator. We will also deploy a frontend, then show how to set up a GitHub webhook so pushes to the repository automatically trigger a Pipeline build.

## 1. Fork the Backend Repository
- Point in the browser logged in to your GitHub account to https://github.com/openshift-for-developers/quarkus-backend.git and click the ``Fork`` button on the top right. This should open a new fork in your GitHub repository. 
- In your new fork, click on the green ``Code`` button and copy the URL of this repository.
  
## 2. Create a New Project for the Noted App
- In the OpenShift Web Console make sure you are in the Developer perspective and open the Topology view.
- In the top-left of the main panel, click on the Project dropdown and click the ``Create Project`` button at the bottom. 
- Name the project "oseXX-noted" where XX is your login number.
- You can put in a display name like "OpenShift Noted Application", and whatever description you would like.

## 3. Deploy the Backend Component
We will build up the deployment in stages. In the first case we will deploy the main branch of the backend (another branch of the backend enables connection to a database).
- Click on the ``+Add`` button on the left sidebar and scroll down to the Git Repository option. 
- Click on ``Import from Git``, and put in the URL of your fork of the Quarkus backend.
- Click on ``Show advanced Git options``, and type ``main`` into the Git reference text box. This ensures the main branch will be used, which does not require a database.
- Scroll down and ensure that Java is selected as the Builder (this should be selected automatically). Change the **Application Name** to ``noted``, and the **Name** to ``quarkus-backend``. 
- Under **Resources** ensure Deployment is selected.
- Under **Pipelines** check the Add pipeline box, and under **Advanced options** uncheck the Create a Route to the Application checkbox - the backend service doesn't need to be accessible externally.
- Click ``Create``
  
## 4. Inspect the Backend Resources
The Web Console and the ``oc`` CLI are both fully featured, so you can inspect your resources using either. 
- Using the Web Console, in Developer perspective, click on ``Project`` on the left hand sidebar. 
- The project overview gives a lot of information on your project. If you scroll down to Inventory, you can see all the resources that have been created by your deployment. There are some Kubernetes resources to manage the current state of the application (Deployments, Pods, PersistentVolumeClaims, Services etc).
- Click on ``Deployments``. Notice OpenShift has automatically deployed an event listener to listen out for Pipeline triggers. If you drill down into the quarkus-backend Deployment you can see a lot more detail. 
- The browser back button will take you back through your steps.
- When you are back on the Project page, click on ``Services`` in the Inventory. OpenShift has put both the event listener and the main quarkus-backend behind a service to manage traffic to dynamically changing Pods for the Deployment.
- The same information is available using ``$oc get all`` from the command line

```
$ oc get all
NAME                                                          READY   STATUS      RESTARTS   AGE
pod/el-event-listener-mk4wp7-5cfd66fcb7-b9hd5                 1/1     Running     0          2m52s
pod/quarkus-backend-1omson-build-b2ngf-pod-mkd9n              0/5     Completed   0          2m7s
pod/quarkus-backend-1omson-deploy-f6wsw-pod-f4fsl             0/1     Init:1/2    0          13s
pod/quarkus-backend-1omson-fetch-repository-nzzl6-pod-dcrrk   0/1     Completed   0          2m51s
pod/quarkus-backend-5b78dd6877-5bbk4                          1/1     Running     0          16s

NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/el-event-listener-mk4wp7   ClusterIP   172.30.206.127   <none>        8080/TCP,9000/TCP   2m52s
service/quarkus-backend            ClusterIP   172.30.106.102   <none>        8080/TCP,8443/TCP   2m51s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/el-event-listener-mk4wp7   1/1     1            1           2m52s
deployment.apps/quarkus-backend            1/1     1            1           2m52s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/el-event-listener-mk4wp7-5cfd66fcb7   1         1         1       2m52s
replicaset.apps/quarkus-backend-5b78dd6877            1         1         1       16s
replicaset.apps/quarkus-backend-5cc48c6cb7            0         0         0       2m52s

NAME                                             IMAGE REPOSITORY                                                               TAGS     UPDATED
imagestream.image.openshift.io/quarkus-backend   image-registry.openshift-image-registry.svc:5000/ose10-noted/quarkus-backend   latest   16 seconds ago

NAME                                                HOST/PORT                                                             PATH   SERVICES                   PORT   TERMINATION   WILDCARD
route.route.openshift.io/el-event-listener-mk4wp7   el-event-listener-mk4wp7-ose10-noted.apps.ose.openshift.conygre.com          el-event-listener-mk4wp7   8080                 None
```
- Note the route at the bottom. Didn't unclick the Create a Route checkbox? This is a route automatically created to the event listener so we can trigger pipeline actions.
- If you want to see the pipeline you can type ``$ oc get pipelines``. 
- The detail of the pipeline can be seen with ``$ oc describe pipeline quarkus-backend``. The first section includes the names you created, some Labels Kubernetes uses to organize itself, and the resource Kind, which is clearly Pipeline.
- If you scroll down to Spec you can see the parameters that are used in the pipeline. This includes your Git repo, Git branch (main) and the container image OpenShift created for you.
- Under Spec you can also find the Tasks subsection. If you scroll through this you will find the three tasks ``fetch-repository``, ``build``, ``deploy`` that are included in your pipeline.

## 5. Deploy the Frontend Component
Remember that we are not going to edit this at all, so it is a straightforward deployment. 
- Click on ``Add+`` then ``Import from Git`` as before.
- Insert the URL https://github.com/openshift-for-developers/nodejs-frontend.git as the Git Repo, and click ``Show advanced Git options``
- Once again we will put **main** as the Git reference. Leave the other boxes as is.
- Ensure Node.js is selected as your builder, put in **noted** as your application and **nodejs-frontend** as the Name. 
- Under **Resources** ensure Deployment is selected. 
- This time we don't really need a Pipeline (since we don't expect any edits in the Repo and couldn't add a webhook to it even if there were),  so make sure the the Add pipeline box under **Pipelines** is unchecked.
- But we do want to make sure the Create a route to the Application box is checked, as this will be the visible frontend of our application.
- Just above the ``Create`` button find the ``Deployment`` link and click on it.
- In the **Environment variables (runtime only)** section add the following two Name/Value pairs (to add the second one you need to click ``+ Add value``)

```
COMPONENT_QUARKUS_BACKEND_HOST quarkus-backend
COMPONENT_QUARKUS_BACKEND_PORT 8080
```
- These environment variables tell the frontend how to find the backend in the local address space. Within the OpenShift cluster, the backend service is available through the DNS hostname ``quarkus-backend`` on port 8080. 
- Click ``Create``. You can watch the process of your build by clicking on the ``Builds`` left sidebar option, then clicking on ``nodejs-frontend`` and selecting the ``Builds`` tab.
- Once both the frontend and backend have "succeeded" as their last run status, you can try the application out!
- Go to the ``Topology`` page and click the Open URL icon on the top left of the frontend app. You will have to click through the "insecure" warnings as before, but eventually you will get to the webpage hosting your deployed app.
- Enter your first note. Perhaps a polite comment about your tutor, or a list of what you would really like to have for lunch.
- Click the ``Submit`` button and your post it note will appear below. You may notice the output is not quite what you would like, so there is a bug in the software that needs to be addressed.

## 6. Add a Pipeline Trigger
Before we fix this bug, we will create a pipeline trigger that will trigger a re-deployment of our application whenever a push is made to the source Git repo. 
Tekton triggers listen out for *webhooks*, which are REST POST requests that you can configure GitHub (and other Git Hosts) to send in response to virtually any activity on the Repo. 
Note that you can configure other elements of the pipeline to include (for example) running a test suite before building. 
Here is something interesting though, in OpenShift 4.10, if you create a Pipeline, a TriggerTemplate is *automatically* created and hooked into it. 
- There's nothing else to do here!
- You do need to know the URL of your EventListener though. Go to  ``Pipelines`` and click on the ``quarkus-backend`` pipeline. On the left of the details page you will see a section called **TriggerTemplates**. The URL under this is the URL of the corresponding EventListener. 

If, for some reason, you need to create you own Pipeline Trigger, here is the process:
- In the Developer perspective on your Web Console, click on ``Pipelines`` and select the quarkus-backend pipeline.
- Click the ``Actions`` menu button on the top-right of the page and select Add Trigger.
- In the **Webhook** section select ``github-push`` from the **Git provider type** dropdown. Notice that there are Git provider types for Bitbucket, GitHub and GitLab. 
- Check that OpenShift has filled in the right parameters for your project (**APP_NAME**=quarkus-backend, **GIT_REPO**=<your backend fork>, **GIT_REVISION**=main). Leve the **IMAGE_NAME**, **PATH_CONTEXT**, **VERSION** and **WORKSPACE** values as they are preset. 
- Click ``Add``.

## 7. GitHub Webhook Configuration
Our pipeline is set up to listen for a webhook from GitHub, we need to config the GitHub Repo to send one in response to a push event. 
- Open the URL of your fork of the quarkus-backend repository that you used for your Deployment. 
- On the list of tabs that starts with ``<> Code`` find the tab marked ``Settings`` and click on it, then select ``Webhooks`` from the left hand sidebar.
- Click the ``Add webhook`` button.
- Copy the EventListener URL (which we found in the last section in the Web Console under Pipelines) into the **Payload URL** window, set the **Content type** to application/json and ensure "Just the push event" is selected for the trigger events. Make sure **Active** is checked and click ``Add webhook``.
  
## 8. Fix the Bug!
Now we are set up to send and receive GitHub webhooks in response to push events. That means any changes that are pushed to the repository will trigger an automatic pipeline build. 
- In GitHub to to your fork of the quarkus-backend and select the ``<> Code`` tab. Click through ``src/main`` then ``java/com/openshift/fordevelopers`` until you see the Post.java file. 
- Click on Post.java, then click on the pencil in the top-right corner to edit the file.
- There are two bugs, in the ``getTitle()`` and ``getContent()`` methods. In both cases a ``StringBuilder`` is used to return a reversed String, whereas we just need to do ``return title;`` for the ``getTitle()`` method, and ``return content;`` for the ``getContent()`` method. 
- Make these changes the scroll down to **Commit changes** and type a git commit message in the top box. Leave everything else the same and click on the ``Commit changes`` button.
- Now go back to your OpenShift Web Console and click on the ``Pipelines`` option on the left sidebar. you should be able to see the ``quarkus-backend`` pipeline working. 
- When the pipeline builds have succeeded, go and refresh the app webpage - you will see that the code now works as expected, without you having to do anything to rebuild or update your deployment. 
- Note that if your build fails (say you check in a syntax bug) then your previous deployment will continue running as usual allowing you to keep your service running while you are fixing your code. 
- Also notice that any existing notes are deleted during the update - that is because this is a stateless app, so an app update will result in losing all state from the previous version. 

## 9. Clean up Resources
- Click on the ``Project`` item in the lefthand sidebar and select Delete project from the ``Actions`` menu on the top right of the page. 
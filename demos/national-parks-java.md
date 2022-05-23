# Deploy a Java Spring Boot App with MongoDB
This closely follows the OpenShift Starter Guide at https://redhat-scholars.github.io/openshift-starter-guides/rhs-openshift-starter-guides/4.9/. There are some updates, including the deployment of the MongoDB which no longer works as specified in the guide. We will use both the Web Console and the Command Line in this tutorial.

## 1. Create a project
- Open the ``Toplogy`` view of the Developer Perspective in the Web Console. 
- At the top left of the page you will see a ``Project`` dropdown. Click on this and select ``Create project``.
- In the **Name** field enter ``oseXX-nationalparks``, with your own login in place of ``oseXX``.
- Enter your own values for the other fields or leave them blank, and click ``Create``.

## 2. The application architecture
The application we will deploy is described in full [here](https://redhat-scholars.github.io/openshift-starter-guides/rhs-openshift-starter-guides/4.9/common-parksmap-architecture.html). In this tutorial we will deploy the parksmap web frontend, and the nationalparks backend with a MongoDB for data persistence. 
On the way we will look at a number of OpenShift features and how they can be used. 

![National Parks Architecture](https://redhat-scholars.github.io/openshift-starter-guides/rhs-openshift-starter-guides/4.9/_images/roadshow-app-architecture.png)

## 3. Deploy the Parksmap Frontend App
The frontend is deployed from the quay.io container repository. 
- In the ``Developer`` perspective select the ``Add+`` option from the left side bar. 
- Scroll down and find the ``Container images`` section, click on that.
- Insert ``quay.io/openshiftroadshow/parksmap:latest`` as the **Image name from external registry**
- This is a Spring Boot app, so in the **Runtime icon** dropdown search for spring-boot and select that icon.
- In the **General** section, set the **Application** to ``national-parks-app``, and the name to ``parksmap``.
- Uncheck the ``Create a route to the application`` checkbox - we will look at how to do that manually later.
- Finally we need to add three labels. Click on the ``labels`` link at the bottom of the page, then type each label as ``name=value`` and press enter after each:
```
        app=national-parks-app
        component=parksmap
        role=frontend
```
- Click the ``Create`` button at the bottom.

## 4. Create a route to the frontend
Clearly you could have done this automatically by checking the box at deployment, but it is also worth seeing how it is done post-deployment. 
- In the `Administrator` perspective, click on `Networking`, then `Routes`
- Click on `Create Route` and put ``parksmap`` in the **Name** field.
- In the **Service** dropdown select the ``parksmap`` service.
- Select the 8080 (TCP) **Target port**
- Check the ``Secure route`` checkbox and choose ``Edge`` from the **TLS Termination** dropdown.
- Leave all other fields as defaults and click ``Create``
- Check your new route by clicking on the route icon to the top right of your app icon in the ``Topology`` view of the Developer perspective.
  - Remember that you will need to click through the various warnings if this is the first time opening this link.
  
## 5. Grant View Permissions to the default Service Account
This parksmap application wants to use the OpenShift API to discover other services that are available. This requires that we grant the ``default`` Service Account the necessary access through the ``view`` RoleBinding.
- In the ``Toplogy`` view, click on the parksmap app to get the right hand side bar up. 
- In the ``Details`` tab click on the link under **Namespace**, then click on the ``RoleBindings`` tab.
- Click ``Create binding``
- Enter ``view`` as the RoleBinding **Name**, then search for the single word ``view`` in the **Role name** dropdown. 
- Select ``ServiceAccount`` in the **Subject** radio buttons.
- Select your current project from the **Subject namespace** dropdown and enter ``default`` as the **Subject name**
- Click ``Create``.
- Now you need to force a redeployment of your application. This is best done from the command line:
```
        oc rollout restart deployment/parksmap
```
- You can head back to the ``Toplogy`` view and click the parksmap app to watch the deployment happen, but it's pretty quick!
- You won't notice any difference in your deployment now, but things will just work as they should when you deploy a backend.

## 6. Deploy the Java Backend
Now we are going to add a Java Spring Boot backend which will provides a REST interface to the parksmap frontend and uses a MongoDB for data persistence.

We will get OpenShift to build and containerize the Java app on the fly using Source-to-image (S2I), which is a tool for building reproducible container images.

- In the Developer perspective click ``+Add`` from the left hand side bar and scroll to the **Git Repository** section and click on ``Import from Git`` 
- In the **Git Repo URL** box insert ``https://github.com/openshift-roadshow/nationalparks.git``
- Click on ``Edit import strategy`` and select ```Builder Image``. 
- Ensure that Java is selected and set the **Builder Image version** to ``openjdk-11-ubi8`` in the dropdown.
- In the **General** section, set the application dropdown to ``national-parks-app`` and set the **Name** to nationalparks.
- Set the **Resources** radio button to ``Deployment``
- Leave the ``Add pipeline`` checkbox unchecked and ensure the ``Create a route to the Application`` box is checked.
- Click the ``Show advanced Routing options`` link and ensure the ``Secure Route`` check box is checked and **TLS termination** is set to ``Edge``.
- Scroll down to the bottom and click the ``Labels`` link, then enter the following labels:
```
        app=national-parks-app
        component=nationalparks
        role=backend
```
- Click ``Create`` to kick off the build process for your backend app.
- You can see the build progress in the ``Builds`` view of the left hand sidebar. 
  - Select the ``nationalparks`` BuildConfig, then choose the ``Builds`` tab. 
  - Your build should be called something like ``nationalparks-1`` and will have a status next to it. Click on the link to get more information on the build. You can see complete build logs and the events list on this page.
- When the Pod is running you can verify your application is working by copying the route link (from the details pane) into you browser and appending ``/ws/info/``.
  - This should give you a simple JSON response from the REST service.  
  - Note that even if the build is complete and the Pod is running, the application may not yet be available. Wait a few seconds and try the link again.

## 7. Connect to a Database
Now we are going to deploy and connect to a MongoDB database which the application can use for data persistence. In this case we will deploy from a container image, then hook it up to the nationalparks backend using a Secret.

- In the ``Developer`` perspective click on ``Add+`` on the left hand sidebar. 
- Scroll down to select  ``Container Images`` for the deployment.
- Select ``Image name from external registry`` and insert the URL ``quay.io/centos7/mongodb-36-centos7``
- Choose the ``MongoDB``  in the **Runtime icon** dropdown.
- In the **General** section set the **Application** to ``national-parks-app`` and the **Name** to ``mongodb-nationalparks``
- Make sure ``Deployment`` is selected in **Resources**
- Uncheck the ``Create a route to the Application`` checkbox.
- Click the ``Deployment`` link at the bottom of the form and enter the following environment variables:

```
        MONGODB_USER: mongodb
        MONGODB_PASSWORD: mongodb
        MONGODB_DATABASE: mongodb
        MONGODB_ADMIN_PASSWORD: mongodb
```
- Click on ``Labels`` and add the following labels:
```
        app=national-parks-app
        component=nationalparks
        role=database
```
- Click ``Create``.
- Note that this stores sensitive information in clear-text as part of your deployment, which is not something you would want to do in a production deployment. OpenShift has other deployment methods and secrets handling that can be used to resolve this issue, but these are outside the scope of this course.
   
There are a number of different ways to connect up a database to an application in OpenShift. Here we will look at creating a Secret object. This is a mechanism OpenShift uses to hold sensitive information like password. OpenShift secrets also provide a very simple way to mount to a running workload, which we will use in this case.

- In the ``Developer`` perspective scroll down to the ``Secrets`` item on the left hand side panel.
- Click the ``Create`` button and select ``Key\value secret``
- In the **Secret name** field enter ``nationalparks-mongodb-parameters``.
- Enter the following key/value pairs: 

```
        MONGODB_USER: mongodb
        DATABASE_SERVICE_NAME: mongodb-nationalparks
        MONGODB_PASSWORD: mongodb
        MONGODB_DATABASE: mongodb
        MONGODB_ADMIN_PASSWORD: mongodb
```
- Click ``Create``
- In the page that comes up, click ``Add Secret to Workload`` in the top right hand corner.
- Select ``nationalparks`` from the ``Select a workload`` dropdown.
- Click ``Save``

## 8. Load the database!
Now we have everything set up to load the data into our MongoDB database. 

- First of all, visit the nationalparks web service to query for data by appending ``/ws/data/all`` to the route for the nationalparks app in your browser.
  - What do you see? Nothing!
- There is a second endpoint which will load data into the database. This time append ``/ws/data/load`` to the route in the browser. You should see a string that tells you something like:
```
    Items inserted in database: 2893
```
- Go back to ``/ws/data/all`` and you will now find your database is fully populated! 

## 9. Now hook up the front end
So far if you go to the route from your frontend, you will see a nice world map, with nothing else on it. That is because this application is querying the OpenShift API to ask about Routes and Services in the porject. If any of them have a **Label** that is ``type=parksmap-backend``, the application will interrogate the endpoints of the backend REST api for map data. 

- Go to the ``Administrator`` perspective and click on ``Networking`` then ``Routes`` in the left sidebar. 
- Find the ``nationalparks`` route and click on it. 
- In the ``Details`` tab click ``Edit`` beside the **Labels** section.
- Add in ``type=parksmap-backend``, press enter, then click ``Save``.
- Now go back to your parksmap frontend and you will find your map beautifully populated with national parks! 


## 10. Add Health Checks
You might remember when we deployed the backend that there was potentially a short delay between when the Pod was running, and the application actually being available to use. This is an example of **Application Health**.  In this section we will add a Readiness probe and a liveness probe to the nationalparks backend to make sure when the Pod says it is ready, it really is ready!

- In the ``Toplogy`` view click on the ``nationalparks`` app to open the right side bar. 
- Click on the ``Actions`` dropdown menu in the top right of the sidebar and select ``Add Health Checks``.
- Click on the ``+Add Readiness probe`` link and leave everything the same except in the **Path** section. In that section insert the following:
```
        /ws/healthz/
```
- Scroll down to the bottom of the form where you see a grey tick and cross. Click on the tick.
- Repeat this procedure with ``+Add Liveness probe``, and use exactly the same **Path** field, leaving all the other entries as defaults.
- Confirm all the changes by clicking ``Add`` at the bottom of the page. 
- These changes cause a new deployment. 

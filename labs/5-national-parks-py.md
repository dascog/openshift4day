# National Parks Python Demo - Web Console
This demo exactly follows the walkthrough at https://docs.openshift.com/container-platform/4.10/getting_started/openshift-web-console.html. It is a helpful demo for OpenShift 4.10 that also includes setting up a database connection.

## 1. Create a Project
- In your Web Console go to the ``Topology`` view in the Developer perspective and click on the ``Project`` submenu on the top-left of the page.
- Click ``Create Project`` and name your project ``oseXX-np`` where XX is your account number. 

## 2. Allocate Permissions for Service Discovery
OpenShift automatically creates a few special "users" called *Service Accounts* in every project. The default service account take responsibility for running the pods, and is injected into every pod that launches. For this app to work, we need the default Service Account to have view permissions into the pods so it can perform service discovery. Without this, the frontend will not know about the backend service. 

- Switch to the ``Administrator`` perspective and scroll down the left side bar to ``User Management``, then click on ``RoleBindings``.
- Click the ``Create binding`` button in the top right corner
- In the **Name** box type ``sa-user-account``, and select your project namespace in the **Namespace** box (i.e. ``oseXX-np``).
- In the **Role name** field search for ``view`` and select that as the Role name.
- In the **Subject** section, choose the ``ServiceAccount`` radio button.
- In the **Subject namespace** field, select your ``oseXX-np`` namespace once again.
- In the **Subject name** field enter ``default``.
- Click ``Create``.
  
## 3. Deploy the Frontend from an Image
- In the Developer perspective click ``+Add`` in the left hand sidebar and scroll down to **Container Images**. Click on that box.
- In the **Image** section select ``Image name from external registry`` and type in quay.io/openshiftroadshow/parksmap:latest
- Ensure the **Application name** is set to national-parks-app and the **Name** is parksmap
- In the **Resources** section select ``Deployment``
- Ensure ``Create a route to the Application`` is checked in the **Advanced Options** section.
- Click the ``Labels`` link and add labels to better identify this deployment. 

```
        app=national-parks-app
        component=parksmap
        role=frontend
```

- Click ``Create``
- Back in the ``Topology`` view you can see your new application deployed! Once it is running click on the route icon on the top right of the app icon - after clicking through the usual warning you should see a world map.

## 4. Spend some time looking through the deployment
- Click on the app and choose the ``Details`` tab on the right hand panel. Scroll down to the **Labels** section and see that your labels have been correctly entered.
- Under the ``Resources`` tab you can see there is one **Pod**, one **Service** and one **Route**. 
- Click on the Pod link.. This brings up a whole lot more details and a lot more tabs. 
- In the ``Logs`` tab you can see the console output from your deployment. This is a good place to look if you have deployment bugs. 
- Click on the ``Terminal`` tab - this gives you a direct login to a container in your Pod. In this case we have just one container in the Pod, so that is the one you are logged into. 
- Enter  ``ps x`` in the command line. You will see that there is an entry ``java -jar /parksmap.jar``, which is the process running your app. If you type ``ls /`` you are taken to the top level directory and you can see the ``parksmap.jar`` application file there. 


## 5. Deploy a Python backend service
- Click on ``+Add`` in the Developer perspective and click ``Import from Git``
- Enter the following as the **Git Repo URL**: https://github.com/openshift-roadshow/nationalparks-py.git
- There are multiple import strategies detected. Click on ``Edit Import Strategy`` and choose ``Builder Image``. Make sure Python is selected as the builder image and choose 3.6-ubi8 as the Builder Image version (it will not work with the default version). Note that the Dockerfile strategy has this builder image version hardcoded. 
- In the **General** section make sure the **Application** is set to ``national-parks-app`` and the **Name** is ``nationalparks``.
- Ensure the ``Create a route to the Application`` checkbox is selected.
- Click the ``Labels`` link and insert the following labels:

```
        app=national-parks-app
        component=nationalparks
        role=backend
        type=parksmap-backend
```

- Click ``Create``


## 6. View the Build Progress
- This app is deployed from the Python source code, so OpenShift needs to build its own Container image to run the code. 
- Click on ``Builds`` in the left hand sidebar and choose the ``nationalparks`` build. 
- From here you can see the progress of the Build - look under the ``Builds`` tab and you will see  build called something like ``nationalparks-1``. Select this and look at the ``Logs`` and ``Events`` tabs to follow the build progress.

## 7. Deploy a Database
Now we will deploy and connect to a MongoDB NoSQL database. This is where the ``national-parks-app`` stores location information. 
We can mark up the ``national-parks-app`` application as a backend for the map visualization tool, ``parksmap`` uses the OpenShift Container Platform discover mechanism to display the map automatically.

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
- Click ``Create``
  
## 8. Connecting the Database
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

## 9. Loading data and displaying the national parks map
Now we have all the components in place, we just need to load data into the database. 
- From the ``Topology`` view, navigate to ``nationalparks`` and click on the route icon on the top-right. 
- The URL should load and state "Welcome to the National Parks data service"
- Append ``/ws/data/load`` onto the URL and press enter, you should see output like:

```
"Items inserted in database: 2893"
```

- From the ``Topology`` view again, click on the route icon for the ``parksmap`` deployment
- You should now be able to see all the National Parks on the map.


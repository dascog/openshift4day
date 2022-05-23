# Deploy a Java Spring Boot App with MongoDB using the Command Line
This closely follows the OpenShift Starter Guide at https://redhat-scholars.github.io/openshift-starter-guides/rhs-openshift-starter-guides/4.9/. There are some updates, including the deployment of the MongoDB which no longer works as specified in the guide. 

## 1. Create a project
- In your command shell enter the command below with your own login substituted for ``oseXX``:
```
        $ oc new-project oseXX-nationalparks``
```
## 2. The application architecture
The application we will deploy is described in full [here](https://redhat-scholars.github.io/openshift-starter-guides/rhs-openshift-starter-guides/4.9/common-parksmap-architecture.html). In this tutorial we will deploy the parksmap web frontend, and the nationalparks backend with a MongoDB for data persistence. 
On the way we will look at a number of OpenShift features and how they can be used. 

![National Parks Architecture](https://redhat-scholars.github.io/openshift-starter-guides/rhs-openshift-starter-guides/4.9/_images/roadshow-app-architecture.png)

## 3. Deploy the Parksmap frontend app
The frontend is deployed from the quay.io container repository.
- First import the image into your imagestreams:
``
        $ oc import-image quay.io/openshiftroadshow/parksmap:latest --confirm
``
- This will print a lot of confirmation out. You can check the image is in place using ``oc get imagestreams``
- Now deploy the image as follows:
```
        $ oc new-app parksmap --name parksmap --labels 'app=national-parks-app,component=parksmap,role=frontend'
```
## 4. Create a route to the frontend
We can create a route with Edge TLS termination as follows:
```
        $ oc create route edge parksmap --service=parksmap
```
- Verify your route is deployed with ``$ oc get route``
- If you copy the route into your browser address bar you can see the frontend in action
  - Remember that you will need to click through the various warnings if this is the first time opening this link.

## 5. Grant View Permissions to the default Service Account
This parksmap application wants to use the OpenShift API to discover other services that are available. This requires that we grant the ``default`` Service Account the necessary access through the ``view`` RoleBinding.
- The following command will do this:
```
        oc policy add-role-to-user view -z default
```
- The ``-z`` syntax is a shortcut that saves us typing the full specification of the service account, which would be ``system:serviceaccount:<project-name>:default``. 
- Clearly this only works in the *current* project.
- Now you need to force a redeployment of your application:
```
        oc rollout restart deployment/parksmap
```
-  You can check the status of the rollout with
```
        oc rollout status deployment/parksmap
```
- You won't notice any difference in your deployment now, but things will just work as they should when you deploy a backend.
## 6. Deploy the Java Backend
Now we are going to add a Java Spring Boot backend which will provides a REST interface to the parksmap frontend and uses a MongoDB for data persistence.

We will get OpenShift to build and containerize the Java app on the fly using Source-to-image (S2I), which is a tool for building reproducible container images.

- To create a new Java application using the Java 11 Builder Image enter the following command:
```
        $ oc new-app java:11~https://github.com/openshift-roadshow/nationalparks.git --name=nationalparks --labels="app=national-parks-app,component=nationaparks,role=backend"
```
- Notice we included in the build command the following labels for the app:
```
        app=national-parks-app
        component=nationlaparks
        role=backend
```   
- We can create a route with Edge TLS termination as follows:
```
        $ oc create route edge nationalparks --service=nationalparks
```
- You can see your route using ``oc get routes``, but we can go one better and open a browser at the ``info`` endpoint with:
```
        $ start chrome https://$(oc get route nationalparks --template='{{ .spec.host}}')/ws/info
```
- This should give you a simple JSON response from the REST service.  
  - Note that even if the build is complete and the Pod is running, the application may not yet be available. Wait a few seconds and try the link again.
  
## 7. Connect to a Database
Now we are going to deploy and connect to a MongoDB database which the application can use for data persistence. In this case we will deploy from a container image, then hook it up to the nationalparks backend using a Secret.

- To deploy the database run the following command:
```
        $ oc new-app quay.io/centos7/mongodb-36-centos7:latest --name=mongodb-nationalparks --labels="app=national-parks-app,component=nationalparks,role=database"
```
- Then add the necessary environment variables with
```
        $ oc set env deployment/mongodb-nationalparks MONGODB_USER=mongodb MONGODB_PASSWORD=mongodb MONGODB_DATABASE=mongodb MONGODB_ADMIN_PASSWORD=mongodb
```
- Note that this stores sensitive information in clear-text as part of your deployment, which is not something you would want to do in a production deployment. OpenShift has other deployment methods and secrets handling that can be used to resolve this issue, but these are outside the scope of this course.

There are a number of different ways to connect up a database to an application in OpenShift. Here we will look at creating a Secret object. This is a mechanism OpenShift uses to hold sensitive information like password. OpenShift secrets also provide a very simple way to mount to a running workload, which we will use in this case.

- Create the Secret with the following
```
        $ oc create secret generic nationalparks-mongodb-parameters --from-literal=MONGODB_USER=mongodb --from-literal=DATABASE_SERVICE_NAME=mongodb-nationalparks --from-literal=MONGODB_PASSWORD=mongodb --from-literal=MONGODB_DATABASE=mongodb --from-literal=MONGODB_ADMIN_PASSWORD=mongodb
```
- The secret can then be added to the deployment using ``oc set env`` as follows:
```
        $ oc set env --from=secret/nationalparks-mongodb-parameters deployment/nationalparks
```

## 8. Load the database!
Now we have everything set up to load the data into our MongoDB database. 

- First of all, visit the nationalparks web service to query for data:
```
        $ start chrome https://$(oc get route nationalparks --template='{{ .spec.host}}')/ws/data/all
```
  - What do you see? Nothing!
- There is a second endpoint which will load data into the database:
```
        $ start chrome https://$(oc get route nationalparks --template='{{ .spec.host}}')/ws/data/all
```
  - You should see a string that tells you something like:
```
        Items inserted in database: 2893
```
- Go back to the first link, refresh, and you will now find your database is fully populated!

## 9. Now hook up the front end
So far if you go to the route from your frontend, you will see a nice world map, with nothing else on it. That is because this application is querying the OpenShift API to ask about Routes and Services in the porject. If any of them have a **Label** that is ``type=parksmap-backend``, the application will interrogate the endpoints of the backend REST api for map data. 

- To see what is currently set in your nationalparks route use the ``oc describe`` command:
```
        oc describe route nationalparks
```
- In the output you will see the list of Labels you applied to this route.
- You can append the new label we want as follows:
```
        oc label route nationalparks type=parksmap-backend
```
- Now go back to your parksmap frontend and you will find your map beautifully populated with national parks!
```
        $ start chrome https://$(oc get route parksmap --template='{{ .spec.host}}')
```


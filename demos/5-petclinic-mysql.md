# Deploy an App with a MySQL Database
In this demo we will deploy an iconic Spring Boot app and use a MySQL database to achieve data persistence. 

# 1. Create a Project and Deploy a Database
- First create a new Project in the Developer perspective, call it oseXX-petclinic (where XX is your account number)
- In this project create a new MySQL instance:
  - Click on ``Add+`` in the left sidebar
  - Scroll to the **Developer Catalog** and click on ``Database``
  - Scroll to ``MySQL (Ephemeral)``. This is a template, which means you will have to put in some parameters
  - Click ``Instantiate Template``
  - Leave the **Namespace** and **Data Service Name** as is, but fill in the following and click ``Create``

```
MySQL Connection Username: petclinic
MySQL Connection Password: petclinic
MySQL root user Password: petclinic
MySQL Database Name: petclinic
```


- In your ``Topology`` view you will now see a MySQL deployment on your screen.
- You can interact with this by clicking on the deployment, then clicking on the link for the running pod. 
- The Pod screen has lots of tabs with different information. 
  - If you click on the ``Environment`` tab you will see the environment variables you just configured, but stored as secrets. 
- Click on the ``Terminal`` tab. This brings us a basic shell into your running Pod. 
- Type in ``$ mysql -u root -h mysql -p`` then enter ``petclinic`` as the root password
- Then type ``mysql> show databases;`` and look at the list of databases - the ``petclinic`` database should be there. Type ``use petclinic;`` to switch to that database. If you do ``show tables;`` there won't be anything there because the database hasn't be populated yet. 

# 2. Deploy the Spring Boot Application
- Click on ``Add+`` in the left side bar and scroll down to the ``Import from Git`` option.
- Put in https://github.com/redhat-developer-demos/spring-petclinic as the **Git Repo URL**
- Click on ``Show advanced Git options`` and put "main" in as the **Git reference** (if you don't do this, the import will fail because it is looking for a "master" branch)
- Scroll down to the import strategies. The import wizard is confused because there are multiple import strategies possible. Click on the ``Edit Import Strategy`` link. Note that if you click on ``Dockerfile`` the import wizard is happy, however the Dockerfile assumes a build has already taken place and the Java jar file is in place, so it won't actually do what we want. Instead we want to get OpenShift to build our app and create a container image using the s2i (Source2Image) utility. So we will choose the ``Builder Image`` option.
- Make sure ``openjdk-17-ubi8`` is chosen as the **Builder Image version** and leave the **Application name** and **Name** as is.
- Under **Resources** make sure ``Deployment`` is selected, leave ``Add pipeline`` unchecked so we can do this deployment from a BuildConfig rather than a Tekton pipeline run. 
- Make sure ``Create a route to the Application`` is checked, then click on the ``Build configuration`` link at the bottom of the screen. This allows us to put in some build and runtime environment variables. Enter the following values using the ``+Add value`` link to put in the second value:

```
SPRING_PROFILES_ACTIVE: mysql
MYSQL_URL: jdbc:mysql://mysql:3306/petclinic
```
- Click ``Create``

# 3. Follow the Build
- Click on ``Builds`` in the left hand sidebar and select ``spring-petclinic``. 
- Click on the ``Builds`` tab and select ``spring-petclinic-1``
- Click on the ``Logs`` tab and you can see the build progressing. When you see "push successful" head back to the ``Topology`` view. 
- Click on the route icon at the top-right of the spring-petclinic app. You will need to click through the normal warning screens until you get to the Petclinic homepage. 
- Click on the ``Veterinarians`` tab to see that your app has data loaded. 

# 4. See that your DB has been Populated
- In the ``Topology`` view click on the MySQL app and then on the Pod link.
- In the Pod, click on the ``Terminal`` tab and login as before: ``$ mysql -u root -h mysql -p`` then enter ``petclinic``
- Type in ``mysql> use petclinic;`` then ``myql> show tables;`` - the tables are populated now. The Petclinic app automatically populates the database on first use. 
- If you want to see the contents of a table, say ``specialities``, just type ``mysql> select * from specialties;``


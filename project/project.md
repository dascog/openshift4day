# OpenShift Project
OpenShift deployment can involve deploying applications you did not write and potentially have no control over the contents. In this project you will attempt a series of deployments. In each case there are two options: you can either attempt to go ahead with the minimal information if you feel comfortable with OpenShift, or you can go through the step by step instructions if you feel less certain. You are encouraged to try out the minimal info first and look at the steps if you get stuck.

## Main Project: 
The project is to deploy a web application consisting of a Java Spring Boot REST backend, a MySQL database and an Angular frontend. The backend will be built from a BitBucket Git repository using source-to-image (s2i). The database will be deployed from a Template and the frontend will be deployed using a Dockerfile build strategy. 

### Minimal Information
For those who feel confident with OpenShift, the minimal information you need is as follows:

1. The bitbucket URL for the backend is https://bitbucket.org/dascog/lol-champions-dsg-jpa.git, the application should be built with the openjdk-11-ubi8 Builder image version.
2. The environment variables the application expects are:
   1. ``DB_HOST``, which should be set to the name of your MySQL service.
   2. ``DB_PORT``, the port for accessing the database, should be set to ``3306``, which is the default port for MySQL deployments.
   3. ``DB_SCHEMA``, the name of the database the application will access, should be set to ``lolchampion``.
   4. ``DB_USER``, defaults to ``root``
   5. ``DB_PASS``, the password for ``DB_USER``, if the user is set to ``root`` this is the root password for the MySQL service.
   6. ``SERVER_PORT`` must be set to ``8080``.
3. It is recommended to use an ephemeral MySQL deployment for the project rather than reply on persistent data.
4. The app has Swagger enabled, so you can access it through the app route with ``/swagger-ui/`` appended.
5. To deploy the frontend you need to fork the repo https://github.com/dascog/lolchampionsui to your own GitHub account and edit the src/environment/environment.prod.ts file to set ``apiUrl`` to the url of your REST backend service. You should deploy using the Dockerfile import strategy. 

### Project in Steps
Here we will describe the project step by step for those who would prefer a more directed approach. This will focus on using the Web Console.

#### 1. Create a new Project 
- Call your project something like ``oseXX-champs``, with ``oseXX`` substituted for your own login.

#### 2. Deploy a MySQL Database
- Use the ``+Add`` option to navigate to the ``Database`` entry in the **Developer Catalog**
- Click on the ``MySQL (Ephemeral)`` Template then click on ``Instantiate Template``.
- Set the following values:
```
        Database Service Name: mysql
        MySQL root user Password: c0nygre
        MySQL Database Name: lolchampion
```
  - You can verify your deployment by opening the Terminal in your MySQL pod and typing ``$ mysql -u root``. This will log you into your MySQL deployment. In the ``mysql>`` command line type ``show databases;`` and verify that your Database Name (``lolchampion``) is in the list. Type ``exit`` to leave. 

#### 3. Create an App from the Bitbucket Repo
- Use the ``+Add`` and navigate to ``Import from Git``. 
- Set the Repo URL to: https://bitbucket.org/dascog/lol-champions-dsg-jpa.git. 
- The default import strategy (Dockerfile) won't work because the Dockerfile presumes the target has already been built. You need to select ``Edit Import Strategy``, then choose a Builder Image with Builder Image version ``openjdk-11-ubi8``.
- Set the **Application name** to ``lol-champions-app`` and the **Name** to ``lol-champions``.
- You *don't* want a Pipeline build in this case. 
- Click on the ``Build configuration`` link at the bottom, and add the following environment variables:
  - If you wish to use a user other than ``root``, you will need to add ``DB_USER`` and ensure you have the right password.
```
        SERVER_PORT: 8080
        DB_HOST: mysql
        DB_PORT: 3306
        DB_USER: root
        DB_PASS: c0nygre
        DB_SCHEMA: lolchampion
```

#### 4. Verify your build
This is a backend REST service for the lolchampion database. You can access it using the Swagger UI as follows:
- Open the route for your app. You should see a **Whitelabel error message**. 
- Append ``/swagger-ui/`` onto your app URL (the final slash is important). This should open the Swagger-UI to interact with the REST client.
- Go to **lol-champion-controller** and try out a ``Post`` request. This way you can post an entry into your database. 
  - Click on ``Post``, then on the ``Try it out`` button on the top right.
  - Fill in entries for ``difficulty``, ``name``, ``role`` - like ``easy``, ``<your-name>``, ``openshift jedi``.
  - Click the ``Execute`` bar and ensure you have a ``201`` response code. 
- You can check that the post was successful by trying out the first ``Get`` request
  - Click on the ``Get`` request followed by **findAll**.
  - Click on the ``Try it out`` button, then the ``Execute`` bar. 
  - You should see your values in the response body!
- You can also see it directly by going into the Terminal of your MySQL Pod and looking at the Database itself:
  - login with ``mysql -u root``
  - at the ``mysql>`` prompt type ``use lolchampion;``
  - then type ``select * from LolChampion;``
  - Your data should appear there!
  
#### 5. Deploy the Frontend
There is an Angular frontend for this backend that allows you to manipulate values. There is a known issue with JavaScript based projects around importing environment variables. To solve this you will have to edit the environment file in the source tree. This means you will first need to fork the repository to your own GitHub account, then work from your new repo. 
- Make sure you are logged in to your GitHub account.
- Point your browser at https://github.com/dascog/lolchampionsui and click on the ``Fork`` button in the top left.
- In your forked version navigate to src/environments/environment.prod.ts. 
- Click the pencil icon to the top right of the file contents to edit the file in the repo.
- You need to replace the apiUrl with the exact apiUrl you use to access the REST controller. If you followed the instructions above it should be as follows: ``https://lol-champions-<your-project-name>.apps.ose.openshift.conygre.com/api/v1/lolchampion/``
- You can check you have it right by pasting the link into your browser and seeing if the REST output displays. Make sure you have the `/` at the end of the URL, the code doesn't check for this and without it deleting and editing won't work.
- Click ``Commit`` and put in a note about what you have done.
- Now deploy the frontend using the ``Import from Git`` option as follows:
  - Set the **Git Repo URL** to your forked and edited repository https://github.com/<your-git-id>/lolchampionsui
  - The Dockerfile import strategy should be selected, stay with this!
  - Set the **Application** to ``lol-champions-app``, and the **Name** to ``lolchampionsui``
  - Don't add a pipeline, but do select ``Create a route to the Application``.
  - Click ``Create``

#### 6. Test your Frontend
- You can test your frontend by clicking on the route icon from lolchampionsui in the Toplogy view of your Web Console. It will be empty, but you can add Champions by clicking the ``Add Champion`` button. If you supply a URL to an image you can also have a mugshot for your champions. 
- Try editing and deleting - you should have full access to all REST CRUD functions. 

## Optional Extra Steps: 
If you have been able to complete the above deployment then here are a couple of extras you could do

### 1. Populate your database
In this part you will use the command line to access your MySQL database and run an sql script to populate the database with LoL Champions. An example sql schema is below, use the demo script to insert it into your database. 
You can include more champions if you like!

```
CREATE DATABASE IF NOT EXISTS lolchampion;

USE lolchampion;

CREATE TABLE IF NOT EXISTS LolChampion (
  id BIGINT NOT NULL AUTO_INCREMENT,
  dateCreated DATETIME NULL DEFAULT NULL,
  difficulty VARCHAR(255) NULL DEFAULT NULL,
  imageUrl VARCHAR(255) NULL DEFAULT NULL,
  name VARCHAR(255) NULL DEFAULT NULL,
  role VARCHAR(255) NULL DEFAULT NULL,
  PRIMARY KEY (id));

  INSERT INTO LolChampion values(1,"2013-06-13","hard","https://static.wikia.nocookie.net/leagueoflegends/images/6/67/Aatrox_OriginalCentered.jpg","Aatrox the Darkin Blade","Juggernaut");
  INSERT INTO LolChampion values(2,"2011-12-14","hard","https://static.wikia.nocookie.net/leagueoflegends/images/f/f1/Ahri_Render.png","Ahri the Nine-Tailed Fox","Burst");
```

### 2. Re-deploy the frontend using a Pipeline build and a webhook
Redo the deployment of the frontend, but this time use a Pipeline build. Set up a webhook from the GitHub repository and demonstrate that it rebuilds when you make a change to your repo.
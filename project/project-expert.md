# OpenShift Project
OpenShift deployment can involve deploying applications you did not write and potentially have no control over the contents. In this project you will attempt a series of deployments. This file includes only minimal information for those who feel they want to test their grasp of the concepts. There is another version of this file that includes a more step-by-step approach if you get stuck. If you conclude the project before time there are some extensions and a stretch goal at the end.

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

### 3. Re-deploy the backend using a Secret to supply the DB environment variables
If you have not already done so, remove any DB-related environment variables from your backend deployment. Then create a Secret with the DB information and apply the secret to your backend workload.

## Stretch Project
Create a template to encapsulate your deployment. The link below should give you enough information to manage it. Start with a template to deploy a single resource, then work up to your existing configuration.
 https://docs.openshift.com/container-platform/4.11/openshift_images/using-templates.html 
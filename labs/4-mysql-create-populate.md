# Create a MySQL database then populate it with a schema
We have looked at creating databases from templates in OpenShift, now we will look at how you can populate those databases. This will be a command line example.

## 1. Create a database from a Template
- Create a new project
- Using the ``Add+`` item, add a MySQL (Ephemeral) database with the following parameters:
```
        MySQL root user Password: c0nygre
        MySQL Database Name: conygre
```
- You can leave all the other parameters either empty or with their default values.

## 2. Download the Schema file to you local machine
The schema.sql file we will use can be downloaded using the following command from your bash prompt:
```
    $ curl -LJO https://raw.githubusercontent.com/dascog/terraform-3day/main/project/schema.sql
```

## 3. Find the Pod running MySQL
You can do this manually by running ``$ oc get pods`` from the command line, or looking at your deployment in the Web Console, however given that we are programmers it would be nice to do it automatically! 
- The OpenShift MySQL template deploys with the label ``deploymentconfig=mysql`` on all of its pods. 
- So you can find the appropriate pods using ``$ oc get pods --selector deploymentconfig=mysql --output=name
- ``
- To use this, we need to strip off the ``pod/`` in which case ``awk`` is your friend! Let's allocate the pod name to a variable, ``mpod``, in which case the whole command is:
```
        mpod=$(oc get pods --selector deploymentconfig=mysql --output=name | awk -F/ '{print $NF}')
```

## 4. Copy your schema file into the running pod
The schema.sql file you downloaded is on your local machine, but to apply it in your pod it needs to be actually inside the pod. The ``oc`` command line provides a helpful subcommand for just this circumstance: ``oc cp``. The command looks like this:
```
        oc cp ./schema.sql $mpod:/tmp/schema.sql
```

## 5. Now run the script!
Remember the ``oc exec`` subcommand? That is exactly what we need to use now. Remember the ``--`` parameter? This just tells ``oc exec`` that everything following should be executed in the pod. The command we need is as follows:
```
        oc exec $mpod -- bash -c "mysql --user=root < /tmp/schema.sql"
```

## 6. Test your database
We can use similar ``oc exec`` commands to see what is going on in our database
- Check which schemas are installed:
```
        $ oc exec $mpod -- bash -c "mysql --user=root conygre -e 'SHOW SCHEMAS;'"
        Database
        conygre
        information_schema
        mysql
        performance_schema
        sys
```
- The ``conygre`` schema is the one we just installed. We can use that to check out the tables available:
```
        $ oc exec $mpod -- bash -c "mysql --user=root conygre -e 'USE conygre; SHOW TABLES;'"

        Tables_in_conygre
        compact_discs
        tracks
```
- Then we can look at one of the tables:
```
        $ oc exec $mpod -- bash -c "mysql --user=root conygre -e 'USE conygre; SELECT * FROM compact_discs;'"
        id      title   artist  tracks  price
        9       Is This It      The Strokes     11      13.99
        10      Just Enough Education to Perform        Stereophonics   11      10.99
        11      Parachutes      Coldplay        10      11.99
        12      White Ladder    David Gray      10      9.99
        13      Greatest Hits   Penelope        14      14.99
        14      Echo Park       Feeder  12      13.99
        15      Mezzanine       Massive Attack  11      12.99
        16      Spice World     Spice Girls     11      4.99
```

## 7. Clean up your Resources
In the command line if you just type ``oc delete project <your-project-name>`` you will release all the resources.



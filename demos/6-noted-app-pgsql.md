# Adding a PostgreSQL Database to the Noted App
Now we will improve the Noted app by configuring a PostgreSQL database to provide data persistence. 

## 1. (Cluster Admin) Add the Sample DB Operators OperatorSource

When we add objects to the cluster, we are actually selecting from catalogues made available from Sources. Go to the developer perspective, topology view and select a Project. Click on the +Add menu item and scrolling to choose all services under the Developer Catalogue. The “Types” down the bottom are the Sources available, including the OperatorSources.

First of all we need to add a new OperatorSource that can provision a correctly configured PostgreSQL database. The construction of the code behind this is beyond the scope of this course, but the installation process is worth seeing. Note that it is a *cluster admin level task*, so you won't be able to do it in your cluster accounts. 

- The ``+`` on the top right of the screen (in both Administrator and Developer Perspectives) allows you to directly create cluster objects using YAML or JSON files. Click on this, and enter the following YAML:

```
  apiVersion: operators.coreos.com/v1alpha1
  kind: CatalogSource
  metadata:
    name: sample-db-operators
    namespace: openshift-marketplace
  spec:
    sourceType: grpc
    image: quay.io/redhat-developer/sample-db-operators-olm:v2
    displayName: Sample DB Operators
```

- Note that we are creating a ``CatalogSource`` object. Now click on ``Create``.  
- Go back to the OperatorHub and search for PostgreSQL. Scroll down until you find an Operator call PostgreSQL Database by the ``Sample DB Operators`` OperatorSource (you can find it by searching ``Bindable``). Click on it to install. Now the PostgreSQL Operator is available to all the users of the cluster.
- If you need to delete a catalogue source, then you can list the ones installed via ``$ oc get catalogsource -n openshift-marketplace``. Then delete the source via 

```
oc delete catalogsource <catalog-source-name> -n openshift-marketplace
```
## 2. Create a PostgreSQL Database from the new OperatorSource
Now that the OperatorSource has been configured, we can continue with your student accounts that have permissions configured to create the necessary services from this source.

- In the Developer perspective open your oseXX-noted Project (top left dropdown menu), then click on the ``+Add`` sidebar item. Scroll to the **Developer Catalog** and click on ``All services``. Choose the ``Other`` link under **All items** and scroll down until you see a tile named ``Database`` from the Operator Backed source. Click on that tile.
- Then click ``Create``
- Ensure the **Name** is set to ``demo-database`` and leave all the other fields as is. Click the ``Create`` button at the bottom.

## 3. Reconfigure the backend to use the PostgreSQL database branch
Remember how we always used the ``main`` branch of the quarkus-backend project? Well, there is a ``pgsql`` branch that is already set up to link in with a PostgreSQL database. 
- In the topology view, click on the ``quarkus-backend`` app to reveal the right side panel. Click on the ``Actions`` submenu at the top-right of the panel and select ``Edit quarkus-backend``
- Click to ``Show advanced Git options`` and change the **Git reference** from ``main`` to ``pgsql``. Then click ``Save``
- Open your browser to your fork of the quarkus-backend project and select the pgsql branch from the dropdown menu on the top left. 
- Navigate to src/main/resources/application.properties and note how the datasource properties will use the environment variables DATABASE_USER, DATABASE_PASSWORD and DATABASE_JDBC_URL if supplied.
- Go back to your OpenShift Web Console - the build should have now completed, and you will notice it has failed! This is no surprise since we have not created the necessary ServiceBinding to facilitate the database connection. (Just because the build failed, doesn't mean your app goes down - the previous build is still up and running).

## 4. Configure a Service Binding
The ServiceBinding object ....

- Create a ServiceBinding object by clicking on the ``+`` at the top of the page and entering the following YAML (making sure to put your own project name in place of ``oseXX-noted``)

```
apiVersion: binding.operators.coreos.com/v1alpha1
kind: ServiceBinding
metadata:
  name: svc-bind-quarkus-database
  namespace: oseXX-noted
spec:
  application:
    group: apps
    name: quarkus-backend
    resource: deployments
    version: v1
  bindAsFiles: false
  mappings:
  - name: DATABASE_JDBC_URL
    value: 'jdbc:postgresql://{{ .postgresDB.status.dbConnectionIP }}:{{ .postgresDB.status.dbConnectionPort }}/{{ .postgresDB.status.dbName }}'
    #value: 'jdbc:postgresql://{{ .postgresDB.metadata.annotations.proxy }}:{{ .postgresDB.spec.port }}/{{ .postgresDB.metadata.name }}'
  services:
  - group: postgresql.baiju.dev
    id: postgresDB
    kind: Database
    name: demo-database
    version: v1alpha1
```
- Click ``Create``
- Now everything functions as necessary! In particular the ``Delete`` functionality now works.

## 5. Inspect the Service Binding
- Click on your ``quarkus-backend`` in the Topology view and go to the ``Resources`` tab.
- Click on the link for the running pod and open the ``Environment`` tab.
- Scroll down and see that the environment variables necessary are all taken from a Secret.
- If you go to the ``Secrets`` submenu and search for this you can see your secret and the values it encapsulates.
  - you can do this in the console with the command following, but in this case all the secret values will be base64 encoded:
  
```
  oc get secret <secret name> -o jsonpath='{.data}'
```

- you can see the details of the ``demo-database`` as well using 
- 
```
  oc describe database demo-database
```

## 6. Persistenace!
- Right, now we can see persistence at work! Ensure you have one or more notes in your app, then close the tab, click on the quarkus backend and scale it to 0 pods. This will destroy all ephemeral memory in the pods. 
- Now scale back up again and open the route. Your saved notes should still be accessible.

## NOTES
- I finally got this working after a very long time. It turns out that a ClusterRoleBinding is required for this to work. The required binding is in the machine-provisioning file. The Admin will need to put it in place.
- There is an alternative Service Binding example in the RedHat documentation: https://docs.openshift.com/container-platform/4.11/applications/connecting_applications_to_services/getting-started-with-service-binding.html


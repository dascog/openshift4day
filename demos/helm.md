# OpenShift Deployment using Helm
Helm describes itself as "the package manager for Kubernetes". Since OpenShift is built on top of Kubernetes, we can use Helm in much the same way. In this demo we will interact with Helm through the ``helm`` cli, but note that you can select and install Helm charts from public repositories using the service catalog included in OpenShift. 
For this demo you need to have a bash shell with the ``oc`` and ``helm`` CLIs installed. If you use the ``Web Terminal`` operator in your OpenShift cluster any user can provision a shell with these command lines installed and ready to go. If not, there are installation instructions for both CLIs under the ``?`` in the console.
## 1. Create a project
- In your command shell enter the command below with your own login substituted for ``oseXX``:
```
        $ oc new-project oseXX-helm``
```
## 2. Setup your Helm Repositories
- It is worth testing the current state of your ``helm`` install first.
```
helm version
helm repo list
```
- If it is not already present, add the bitnami repo from which we will get an nginx distribution.
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
- With ``helm search hub`` you can search the artifact hub (https://artifacthub.io/) for helm charts from dozens of different repos. 
```
helm search hub nginx
```
- This clearly has a large number of resources that are based on or linked to nginx! In fact we will use a Helm chart provisioned by the bitnami repo we added, which can be searched using ``helm search repo`` as follows:
```
helm search repo bitnami/nginx
```
- We want to use the ``bitnami/nginx`` chart in our deployment.

## PART 1 - Helm Install from a Repository
### 1.1. Install the Nginx chart
- The Helm install statement is very simple. The ``--set service.type=ClusterIP`` exposes the service on a cluster-internal IP, so the service is only reachable from within the cluster (https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)
```
helm install my-nginx bitnami/nginx --set service.type=ClusterIP
```
- If you go to your cluster console and open the ``oseXX-helm`` project, you will see the nginx deployment.
### 1.2. Check your Helm and OpenShift resources
- You deployment has resource registered with both Helm and OpenShift. You can view your Helm resources as follows:
```
helm ls
```
- Using the ``oc`` CLI you can also verify that the pods are running as expected:
```
oc get pods
```
### 1.3. Expose the my-nginx service via an openshift route
- When we installed this chart we used the ``ClusterIP`` service type to define an internally accessible route. OpenShift routes are a convenient way to supply an externally accessible route to the nginx service:
```
oc expose svc/my-nginx
```
- Verify the route has deployed correctly
```
oc get routes
```
- You can test the link by either copying or pasting into a browser tab, or accessing the link in your console. 
### 1.4. View the Helm Releases for this app
- In the developer perspective of your console, click on the ``Helm`` link on left sidebar. This will display the release information for you nginx deployment.
  
### 1.5. cleanup
- To cleanup this deployment we have to remove the resources from the two points of creation - the helm install and the route created using ``oc expose``:
```
helm uninstall my-nginx
oc delete route my-nginx
```

## Part 2 - Deploying with Helm Charts
The ``oseXX-helm`` project should not be clear, we will use it for a second deployment of nginx, this time using a Helm Chart from our local filesystem.
### 2.1 Create a local Helm Chart
- For this part you need to create a suitable subdirectory in your local filesystem to contain you helm chart. We will use ``~/helm``. 
```
mkdir ~/helm
cd ~/helm
```
- The ``helm create`` command prefills a chart with default values:
```
helm create my-chart
cd my-chart
```
### 2.2 Review the chart contents
The create command generates a skeleton Helm Chart with an NGINX image as example. It contains the following files and directories

| File | Description |
|------|-------------|
| Chart.yaml | A YAML file containing multiple fields describing the chart |
| values.yaml | A YAML file containing default values for a chart, those may be overridden by users during helm install or helm upgrade. |
| templates/NOTES.txt | text to be displayed to your users when they run helm install |
| templates/deployment.yaml | a basic manifest for creating a Kubernetes deployment |
| templates/service.yaml | a basic manifest for creating a service endpoint for your deployment |
| templates/_helpers.tpl | a place to put template helpers that you can re-use throughout the chart |

### 2.3 Look more closely at Chart.yaml
- you can do this in your command line using the ``cat Chart.yaml`` command. 
- Chart.yaml contains version of the package and appVersion that we are managing, typically this can be refered to a container image tag. 

### 2.4 Fill chart with custom values
- This example uses the  Helm Template templates/deployment.yaml which describes a Kubernetes Deployment for our app. The container image used for the app is specified by the following structure (spec.containers.image) which will populate with the values specified in ``values.yaml``:
```
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

- Note that if ``.Values.image.tag`` is unspecified, the appVersion from Chart.yaml is used as the image tag

- Now we can edit values.yaml to add the image.repository variable which will define the container image for our chart.
- Using a command line editor (use ``nano values.yaml`` if you are not familiar with command line editing) change the repository entry as follows to ensure we use the bitnami image:
  
```
repository: bitnami/nginx
```
- We will also define the tag to use for this container image. Edit the ``tag`` entry below the ``repository`` entry you adjusted above:
```
tag: latest
```

### 2.5 Install your chart
- Install your custom Helm Chart from the local folder.
```
cd ~/helm
helm install my-chart ./my-chart
```
- This will install NGINX as we did in the previous section. You can observe your resources here as we did then:
```
oc get pods
helm ls
```

## Part 3 - Revisions
When a Helm Chart is installed on OpenShift, a release is published into the cluster that we can control in terms of upgrades and rollbacks.
To change something in any already published chart, we can use helm upgrade command with new parameters or code from our chart.

### 3.1 Add an OpenShift Route to your chart
- Change into the templates directory:
```
cd ~/helm/my-chart/template 
```
- Now create a new resource file called ``routes.yaml``. The file content is given below and the command will create the necessary file and input the content. Just copy and paste the command into your CLI:
```
cat <<- EOF > routes.yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: {{ include "my-chart.fullname" . }}
  labels:
    {{- include "my-chart.labels" . | nindent 4 }}
spec:
  port:
    targetPort: http
  to:
    kind: Service
    name: {{ include "my-chart.fullname" . }}
    weight: 100
  wildcardPolicy: None
EOF
```
### 3.2 Run helm upgrade
- This will publish a new revision containing a my-chart Route:
```
helm upgrade my-chart ./my-chart
```
- Verify new Route from Terminal:
```
oc get routes
```
- Verify new Revision:
```
helm ls
```
- The new route and revision can also be verified in the console.

### 3.3 Upgrade and Rollback
In this section we will run an update that overrides values in values.yaml using the ``--set`` command line argument. In particular we will change the default ``image.pullPolicy`` from ``IfNotPresent`` to ``Always``.

- The ``helm upgrade`` command is the same as above, except now we explicitly indicate the new  ``image.pullPolicy`` through the ``--set`` option:
```
helm upgrade my-chart ./my-chart --set image.pullPolicy=Always
```
- Check the deployment to see that the change has been implemented:
```
oc get deployment my-chart -o yaml | grep imagePullPolicy
```
- And see the current Revision:
```
helm ls
```

- Now that our new release is published and verified, we can decide to rollback to previous version if we need to, and this is possible with ``helm rollback`` command. There is an option of testing that the rollback will proceed successfully by using the ``--dry-run`` option. This will not change the current version but will indicate if a rollback to version 2 would complete successfully:
```
helm rollback my-chart 2 --dry-run
```
- If the dry run is successful, then do an actual rollback to revision 2:
```
helm rollback my-chart 2
```

- Verify the imagePullPolicy is rolled back to Revision 2 containing IfNotPresent Policy:
```
oc get deployment my-chart -o yaml | grep imagePullPolicy
```
### 3.4 Uninstall
Uninstall will clean everything now, there's no further need to delete manually the Route as in first section, since the Helm Chart is now managing that resource:
```
helm uninstall my-chart
```
# Deployment Strategies Example
We will use a simple deployment to demonstrate the rolling and recreate update strategies with a *DeploymentConfig* project, then with a *Deployment* based project.
This is based on the examples at https://github.com/openshift/origin/tree/master/examples/deployment

## 1. Create the Project
- ``oc new-project oseXX-depl``
- You can do the same by clicking on ``Project`` in the Developer Perspective of the Web Console

## 2. Deploy the application
- ``oc create -f deploy-example.yaml``
- Check that the application is running using the Web Console.
- Note that the Update strategy is set to Recreate.

## 3. Trigger an Update using the Recreate strategy
- Make sure you are in the ``Details`` tab of the right sidebar
- Under the ``Actions`` menu click on ``Start rollout``
- See that the Pods scale down to 0, then back up to 2.
- Click on the app to close the side bar and open it again.
- Note that the Latest Version is now 2.

## 4. Change the Update strategy to Rolling
- Under the ``Actions`` menu choose ``Edit DeploymentConfig``
- Change the **Strategy type** to Rolling.
- Set the following:
  
```
        Timeout:10
        Maximum number of unavailable Pods: 50%
        Maximum number of surge pods: 50%
```

- Click ``Save``
- Back in the ``Topology`` view, click on the ``Actions`` menu and select ``Start rollout``
- Observe that the current deployment stays running while the new pods are added
- Observe that the version number is now 3

## 5. Deployment strategies with a Deployment based project
- Things are a little different if your project implements a *Deployment* rather than a *DeploymentConfig*.
- In this case there is no need to trigger a new rollout with ``Start rollout`` because the deployment automatically rolls out changes that are detected unless explicitly paused.
- The deployment strategy is adjusted by the ``Edit Deployment`` option in the ``Actions`` menu. 
- To trigger an update, you need to make a change, for example, edit the labels to add a version number.

# Creating Persistent Storage for the Image Uploader
This lab follows directly on from 1-image0-uploader-deployment.md and presumes you have concluded up to step 5 of that lab. We will use the PersistentVolume allocated in the cluster to attach persistent storage to this app using a PersistentVolumeClaim. 

## 1. Verify that your app doesn't store data.
- In the ``Topology`` view of the ``Developer`` perspective click on the ``image-uploader`` app. You should have 1 Pod running. 
- Open the route for the app and upload an image. 
- In the ``Resources`` tab of the right side bar click on the link for the running pod, then click on the ``Terminal`` tab.
- In the terminal type ``ls uploads`` - you should see your image file in that directory.
- Now go back to the ``Topology`` view and scale your app to zero Pods using the arrows next to the Pod in the ``Details`` tab.
  - There is a bug in OpenShift so you might not have those arrows with some deployments. If that is the case, fire up your command line (remember you can copy the login command by clicking on your username in the top right corner of the page). The ``oc`` command for scaling the deployment is just ``oc scale --current-replicas=1 --replicas=0 deployment/image-uploader`` to scale down, and `oc scale --current-replicas=0 --replicas=1 deployment/image-uploader`` to scale up. 
- When the new pod is scaled up, go back into the new Pod terminal and type ``ls uploads`` again. Your image file won't be there. No data persistence!
  
## 1. Add a PersistentVolumeClaim for the Image Uploader Project
- Go to the ``Administrator`` perspective and click on ``Storage`` in the left side bar.
- Choose ``PersistentVolumeClaims`` and click on the ``Create PersistentVolumeClaim`` button.
- Ensure the correct project is selected (top-left of the screen), and change the name to **oseXX-pvc**
- The access mode cannot be changed because of our cluster deployment on AWS.
- In **Size** select MiB and put in ``100`` (we don't need much)
- Leave the Volume mode on ``Filesystem``
- Click ``Create``

## 2. Create a Volume for the App
- In the ``Developer`` perspective click on ``Project`` in the left side bar.
- Under the **Inventory** section, click on the ``Deployments`` link
- Choose the ``image-uploader`` deployment
- Click on the ``Actions`` menu on the top-right
- Select the ``Add Storage`` option.
- Select the PVC you created above.
- For the **Mount path** put in ``/opt/app-root/src/uploads``
- Click ``Save``

## 3. Test out your Persistent Storage
- In the ``Topology`` view click on the route shortcut on the top-right of your app icon.
- Browse, select and upload an image
- Now look at the resources for your app in the ``Topology`` view and click on the running Pod.
- Choose the ``Terminal`` tab, and type ``ls uploads`` - your image should be in the list.
- Return to the ``Topology`` view and click on the ``Details`` tab for the Image Uploader.
- Scale the Pods down to 0 and wait for a few seconds. Verify the app no longer works by clicking on the route shortcut.
  - Note that if your pod doesn't have up and down arrows you can still scale the Pod through the ``oc`` cli. Use the command ``oc scale --current-replicas=1 --replicas=0 deployment/image-uploader`` to scale down to zero, then ``oc scale --current-replicas=0 --replicas=1 deployment/image-uploader`` to scale back up again. This is an OpenShift bug in the Web Console.
- Now scale the app back up to 1. When the pod is running, click on the route shortcut and see that your previously uploaded image is still available.
- You can check using the pod terminal that your image is still visible within the pod with ``ls uploads``. 


## 9. Clean up Resources
- Click on the ``Project`` item in the lefthand sidebar and select Delete project from the ``Actions`` menu on the top right of the page. 
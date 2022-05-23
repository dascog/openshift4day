# Creating Persistent Storage for the Image Uploader using the Command Line
This lab follows directly on from 1-image0-uploader-deployment-cli.md and presumes you have concluded up to step 5 of that lab. Here we use the PersistentVolume allocated in the cluster to attach persistent storage to this app using a PersistentVolumeClaim. 

## 1. Verify that your app doesn't store data.
- Ensure that you are in the right project space: ``$ oc project`` will tell you the current project you are in, if you need to change you will need either ``$ oc project oseXX-imageuploader`` or ``$ oc project image-uploader-cli`` (or whatever you named the project from lab 1).
- Type ``$ oc get pods`` in the command line, you should see something like:
```
    $ oc get pods
    NAME                             READY   STATUS      RESTARTS   AGE
    image-uploader-1-build           0/1     Completed   0          4d
    image-uploader-89b7d5d4f-wbdx8   1/1     Running     3          3d23h
```
- You can verify that you app does not have persistent memory by the following steps
  - Open your app route with ``$ start chrome https://$(oc get route image-uploader --template='{{ .spec.host}}')``
  - Upload an image in the web UI. 
  - Verify the image is in your Pod:
```
      $ oc exec <your-running-pod-name> -- ls uploads
      lost+found
      stevejobspie.jpg
```
  - Scale the deployment down to zero pods: ``$ oc scale --current-replicas=1 --replicas=0 deployment/image-uploader``
  - Wait 30 seconds or so, then scale back up to one pod: ``$ oc scale --current-replicas=0 --replicas=1 deployment/image-uploader``
  - Use ``$ oc pods`` to get your new running pod name
  - Once the pod has status ``Running``, run the ``$ oc exec <your-running-pod-name> -- ls uploads`` again to see that your image is now not present.

  
## 2. Add a PersistentVolumeClaim for the Image Uploader Project
- Copy the following code into a file called ``image-uploader-pvc.aml``:
```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: image-uploader-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 100Mi
```
- Now create the PersistentVolumeClaim:
```
    $ oc create -f image-uploader-pvc.yaml
```
- The PersistentVolumeClaim will make a claim to any available PersistentVolume in the cluster that is suitable for its request. Attach to the running deployment at the mountpath ``/opt/app-route/src/upload`` in the pods with the following command:
```
    $ oc set volume deploy/image-uploader --add --type=persistentVolumeClaim --mount-path=/opt/app-root/src/upload--claim-name=image-uploader-pvc
```
- This will result in a new deployment. 

## 3. Test out your Persistent Storage
- Follow the sequence above to upload an image to your app. 
- Now scale the deployment down to 0 pods using ``$ oc scale --current-replicas=1 --replicas=0 deployment/image-uploader``
- After a short while, scale the deployment back up to 1: ``oc scale --current-replicas=0 --replicas=1 deployment/image-uploader``
- Use ``$ oc get pods`` to find the name of the running pod.
- Then run the exec command:
```
    $ oc exec <your-running-pod-name> -- ls uploads
```
- You should see your image filename in the pod!

## 9. Clean up Resources
- The project and all its resources can now be deleted using ``$ oc delete project <project-name>``
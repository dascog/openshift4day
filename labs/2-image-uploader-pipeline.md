# Deploy the Image Uploader with a Pipeline
In this lab you will once again deploy the simple Image Uploader we saw previously, but this time you will deploy your own fork of the GitHub repository, and you will use a Tekton Pipeline to build it.

## 1. Fork the GitHub Repo
- Point your browser to the original GitHub repo at  https://github.com/OpenShiftInAction/image-uploader/ and click the ``Fork`` button to create your own fork of the repository.

## 2. Create a new project
- In the ``Topology`` view of the ``Developer`` perspective click on the ``Project`` drop-down at the top-left of the page. 
- Click on the ``Create project`` button at the bottom. 
- Name your project ``oseXX-pipeline`` and choose a Display Name and Description. 

## 3. Import your GitHub repo
- In the ``Developer`` perspective click on ``Add+`` in the lefthand sidebar
- Scroll down to find the **Git Repository** section and click on ``Import from Git``.
- Put the URL of your fork of the image-uploader project as the Git Repo URL.
- In the **General** section put in ``image-uploader-app`` as the Application Name, and ``image-uploader`` as the Name.
- In the **Pipelines** section select the ``Add pipeline`` checkbox.
- Leave everything else as is and click the ``Create`` button.

## 3. Observe the Build process
- Click on the ``Builds`` item in the lefthand sidebar - notice that this time there are no builds.
- Click on the ``Pipelines`` item and watch the build progress of your pipeline.
- When you build has succeeded go to the ``Topology`` view. Observe that your app has been joined by a ``Triggers`` app which contains the EventListener for your build Pipeline.

## 4. Find the EventListener URL
- Go to the ``Projects`` view from the left hand sidebar and click on your project. 
- Find the **TriggerTemplates** section on the right hand side and note the URL.

## 5. Create a GitHub webhook
- In your GitHub fork of the image-uploader project click on the ``Settings`` tab at the middle right of the page. 
- Select ``Webhooks`` from the left hand menu and click on the ``Add webhook`` button at the top right of the page.
- The **Payload URL** is the EventListener URL you found in Step 4.
- Set the **Content type** to ``application/json``
- Ensure that the "Just the push event" option is selected and click ``Add webhook``.

## 6. Make a change to your repo and watch the Pipeline work!
- In your GitHub repo click on the ``<> Code`` tab, and click on the index.php file.
- Edit the file by clicking the pencil icon on the top right. 
- Make a visible change, for example, on line 57 there is the main header of the app. Change the text to be ``Excellent Image Library with ``, or somesuch.
- Make an appropriate commit comment at the bottom and click ``Commit changes``.
- Back in the Web Console click on the ``Pipelines`` view and see the progress of the pipeline build you have trigger with your commit. 
- Once it is completed, open the route from the ``Topology`` view and see your changes.

## 7. Clean up resources
- Once you have finished, click on the ``Project`` view and select Delete Project from the ``Actions`` dropdown on the top right.
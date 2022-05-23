# Deploy a Project From GitHub
In this lab you will find your way around the Web Console by deploying a PhP project from GitHub. This app will upload images and to the webpage and show the uploaded images in a gallery.

## 1. Create a New Project
- Log into the Web Console, select the Developer perspective and click on the ``Project:`` drop-down menu. Click on the ``Create Project`` button at the bottom of the menu.
- Put in ``oseXX-imageuploader`` as the Name of the project, and choose a suitable Display name and Description.

## 2. Add the App to your project
- Click on the ``+Add`` item in the left sidebar, and select the ``Import from Git`` option. 
- The **Git Repo URL** is https://github.com/OpenShiftInAction/image-uploader/
- There is only one branch (master) in the Repo, so you don't need to fill in the Git reference. 
- Check that PHP is selected as the Builder.
- In the **General** section set the Application to ``image-uploader-app``, and the Name to ``image-uploader``.
- Leave everything else as default, checking that the Create a route to the Application checkbox under **Advanced Options** is ticked, and click ``Create``.

## 3. Check your build progress
OpenShift is now going about the process of cloning the GitHub project, building a container image and deploying it. 
- Select ``Builds`` from the left hand sidebar and choose ``image-uploader``.
- Choose the ``Builds`` tab from the top bar - the Status will tell you when your build is complete. The ``Logs`` tab will show you exactly what is happening as your build progresses.
- In the ``Topology`` view you should be able to see your app. When there is a green tick on the bottom left your build is complete. Click on the route link (top right) to see your app running. Try to load an image file and see what happens. What happens when you try to load a file that is not an image?

## 4. Scale your app up and down
- In the ``Topology`` view click on your app and look at the ``Details`` tab in the right hand sidebar. Currently your app is running in one Pod. Try scaling the number of Pods up and down using the up and down arrows. This is all transparent to your users, but gives you control over the resources you make available. 

## 5. Go through the menus to see what your deployment has put in place
Familiarize yourself with the Web Console. See what resources have been created as part of your Build process.

## 6. Remove your Project
This is a single application project, so once you have finished it is a good idea to clean up your resources!
- Click on ``Project`` in the left hand sidebar. Click on the project you just created.
- Click on the ``Actions`` menu on the top left of the page and select Delete project. This will require you to enter your project name as confirmation you want to delete it. 


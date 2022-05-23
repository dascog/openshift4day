# Logging in to OpenShift and Creating a Deployment
To complete this lab you will need:
1. a login to an openshift cluster
2. a Linux shell (Git Bash is fine)
3. your own GitHub account.

# 1. Log in to the web console
Our OpenShift cluster is already up and running. It is a standard AWS install. You can find it by pointing your browser to http://console-openshift-console.apps.ose.openshift.conygre.com
- The OpenShift console uses HTTP rather than HTTPS so we need to click through some warnings!
- On the "Your connection is not private" page, click on the ``Advanced`` button, then click on the ``Proceed to console-openshift-...`` link at the bottom of the page
- You may get a second warning page as it links you to oauth-openshift...., do the same thing with Advanced, then click on the link.
- On the login page, click on ``my_htpasswd_provide``
- Put in the username and password you have been supplied with.
- If all goes to plan you are now in the console!

# 2. Download the OpenShift command line tool
We will be switching between the console and the command line, so you need to have both up and running. 
- You will need a Linux shell open in front of you (Git Bash does well on windows).
- Click on the ``+Add"` item in the sidebar of the Developer view. You should see a "Getting Started" page.
- In the middle of the page there is a link to command-line tools. Click on this
- Download the appropriate binary for your platform. 
- Once again you will be warned about your insecure connection - click through using Advanced and click the download link. 
- When the download is complete you will have a zip file in your downloads directory with the appropriate executable inside. 
- Extract this executable, and place somewhere in your path. To know what directories are in your path, type ``echo $PATH`` in your Linux shell. For a Linux environment, the ``~/.local/bin`` directory is often a good choice. For Windows you may need to create a ``bin`` directory in your home directory ``C://Users/<your-username>/bin`` and copy the ``oc.exe`` file there. 
- When it is in place, type ``oc version`` from your Linux command line to verify it is working OK.


# 2. Log in to the oc command line
- Back in the web console, click on your username in the top right corner of the screen and select "Copy login command". This opens a new tab at the login screen. 
- Go through the console login steps we did previously until you get to a blank page with just "Display Token" in the top left of the page.
- Click on that link and copy the command under "Log in with this token"
- Paste that into your Linux command line and press enter.
- *IF* you get a login error saying the server uses a certificate signed by unknown authority, copy the command again and include the option ``--insecure-skip-tls-verify`` before the --server option.
- to check you are logged in correctly, type ``oc whoami``
  
# 3. First steps around the console
Go back to the web console and have a look around. There isn't currently much to see because you don't have anything deployed or running.
- The left hand side has a menu bar (click on the three horizontal lines at the top left if it's not visible). The first entry has either Administrator or Developer visible. These are the two views you have into your console. If you switch between them you will see the options change in the sidebar. For the moment we will focus on the Developer view. 
- To do anything at all, you will need to create a project. 
- Click on the ``+Add`` side bar option, then click on ``Create a new project``
- In the Create Project dialogue, put ``oseXX-hello`` (where ``XX`` is your number) for the project name. For the display name put whatever you like (it will be visible to me!), and the same for the description.
- Now click on the Topology link and you will see the project is automatically selected in the window. There are no resources there because you haven't deployed any yet. 
- Click on the ``+Add`` sidebar item.You can see there are a lot of options for how you can get code to run on your cluster. 
- For now, find the panel titles "Create applications using samples" and click on "View all samples".
- There are a lot of sample applications here, mostly of the "Hello world" variety. Choose one of the "Basic" samples
- See how the sample pre-fills a Git repo URL and a Name for our component. 
- Click create.
- Go to the Topology view in the sidebar. There are two views of your topology - a graphical view, which has a circle with some icons, or a list view, which gives a list of your deployments. You can switch between them by clicking the splodge / bullet list in the top right under your user name.
- In either view click on the application and a panel will open on the right hand side. There is a lot of detail about your application here. Assuming the build goes to plan, after a while you will have a single circle representing a single running pod under the details menu on the right hand panel, and there will be a green tick on the bottom left of your app icon. 
- If you click on the arrow icon at the top-right of your app icon you should get a web page opening up and saying "Hello World!" or something like that. 
- There is also a link to the GitHub page if you want to examine the code. 
- Congratulations! You have deployed your first application to an OpenShift Cluster!
- To clean up your deployment, click on the app link (for these apps often called sample-app). A right sidebar will option and has an "Actions" menu on the top right. Click on it, and select "Delete Application". Confirm delete by typing the app name in the box. 

# 4. Changing a deployment
To deploy your app, you didn't need to compile anything, or containerize your app - OpenShift did all this for you! OpenShift can also integrate your changes directly from a Git repo.
- You will need to deploy an application from a repo you have write permissions to. A GitHub account is free and quick to sign up (https://github.com/signup), but *don't* use your work email address for it! 
- Now go through the steps to add a sample application above and select the Basic Node.js project, but don't create it.
- Instead, copy the repository link ( https://github.com/nodeshift-starters/devfile-sample.git) into the browser you logged into GitHub. On the top left of the page there is a ``Fork`` button. Click it and you should be offered to fork this project to your own GitHub repo. Make some kind of description to help you remember what you are doing!
- Now you should be pointed to your fork of the repository. Click on the green "Code" button and copy the repo URL.
- Back in the OpenShift console, cancel your sample deployment and click on the ``+Add`` sidebar button again. Find the "Import from Git" option and click on that.
- Copy the git URL of your fork into the url window and click "Create".
- Once it is running you can verify by opening the OpenShift route link in the top left of the app icon.
- Now, in your GitHub fork, click on the file ``server.js`` and then click on the pencil on the top right to edit the file. Scroll down to the ``app.get`` method. Change the ``res.send`` message to your own personalized message.
- Click "Commit Changes" down the bottom. 
- Back in the OpenShift console, click on the ``Builds`` item in the left side-bar. You already have a BuildConfig for your sample app - OpenShift created it automatically. Click on the three dot menu on the right hand side and select ``Start build``
- Go back to the Topology page and you will see the building (recycle) icon on the bottom left of your app icon showing you a build is in process. This is pulling your updated code from GitHub and rebuilding you application. Once the build is complete, click on the Route icon and you will see your updated code has now deployed.
- You have just done the key steps to deploying your code using OpenShift. Once a BuildConfig is in place and your app is deployed, OpenShift will fade into the background and you can concentrate on developing your code.
  
## 5. Clean up Resources
- Click on the ``Project`` item in the lefthand sidebar and select Delete project from the ``Actions`` menu on the top right of the page. 

# Lab 2: Running Docker Images in IBM Containers

> **Difficulty**: Intermediate

> **Time**: 30 minutes

> **Tasks**:
>- [Prerequisites](#prerequisites)
- [Task 1: Tag the images and upload to IBM Containers Bluemix](#task-tag-the-images-and-upload-to-ibm-containers-bluemix)
- [Task 2: Verify security vulnerabilities](#task-2-verify-security-vulnerabilities)
- [Task 3: Run your web app on Bluemix](#task-3-run-your-web-app-on-bluemix)

## Prerequisites

Prior to running this lab, you must have a Bluemix account and setup the IBM Containers command line locally.  Instructions are available in [Prerequisites](https://github.com/bainsy88/containers-denmark/blob/master/0-prereqs.md).  You must have also completed [Lab 1](https://github.com/bainsy88/containers-denmark/blob/master/1-Intro-to-IBM-Containers-and-Docker.md).

If you are using a trial account, ensure that you have removed all non-essential containers, as these will impact your quota whether they are running or not. You can remove containers by using the Bluemix UI or IBM Containers CLI.

## Task 1: Tag the images and upload to IBM Containers Bluemix

You will tag the previously downloaded images with your unique namespace so you can upload those images to your public registry hosted on Bluemix.

1. Make note of your namespace:  
       
        $ cf ic namespace get    
        ibm_containers_demo_eu

2. Tag your MongoDB image.  Remember to use your namespace from the first step to replace `[NAMESPACE]` in the tag and push commands below.  The namespace tag ensures that the image is uploaded to your private registry in the Bluemix cloud.

  List your local images:
   
        $ docker images
        REPOSITORY                                                    TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
        mongo                                                         latest              202e2c1fe066        7 days ago          261.6 MB
        sdelements/lets-chat                                          latest              2409eb7b9e8c        4 weeks ago         241.5 MB
      
  Tag your MongoDB image in a Bluemix-compatible format:  
  
        $ docker tag mongo registry.eu-gb.bluemix.net/[NAMESPACE]/mongo
  

  Tag your Let's Chat image in a Bluemix-compatible format:
 
        $ docker tag sdelements/lets-chat registry.eu-gb.bluemix.net/[NAMESPACE]/lets-chat


  List your images again, now showing the newly tagged image:  

        $ docker images
        REPOSITORY                                                    TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
        mongo                                                         latest              202e2c1fe066        7 days ago          261.6 MB
        registry.eu-gb.bluemix.net/ibm_containers_demo_eu/mongo       latest              202e2c1fe066        7 days ago          261.6 MB
        sdelements/lets-chat                                          latest              2409eb7b9e8c        4 weeks ago         241.5 MB
        registry.eu-gb.bluemix.net/ibm_containers_demo_eu/lets-chat   latest              2409eb7b9e8c        4 weeks ago         241.5 MB
  

  Note that the `IMAGE ID` column did not change for the MongoDB or Let's Chat image.  Since we are not modifying the image, but rather simply giving it another name, the `IMAGE ID` stays the same and allows us to reuse the existing container image as-is.
  

2. Now that your images are tagged in the correct format, you can push them to your private registry on Bluemix.  This allows the IBM Container service to run your container images on the cloud.

  Push your MongoDB image to your Bluemix registry:  

        $ docker push registry.eu-gb.bluemix.net/[NAMESPACE]/mongo
        The push refers to a repository [registry.eu-gb.bluemix.net/[NAMESPACE]/mongo] (len: 1)
        Sending image list
        Pushing repository registry.eu-gb.bluemix.net/[NAMESPACE]/mongo (1 tags)
        Image 68e42ff590bd already pushed, skipping
        Image b4c4e8b590a7 already pushed, skipping
        f037c6d892c5: Image successfully pushed
        1a64ad3ccff1: Image successfully pushed
        ...
        a08422dd6a11: Image successfully pushed
        202e2c1fe066: Image successfully pushed
        Pushing tag for rev [202e2c1fe066] on {https://registry.eu-gb.bluemix.net/v1/repositories/[NAMESPACE]/mongo/tags/latest}


  Push your Let's Chat image to your Bluemix registry:  
 
        $ docker push registry.eu-gb.bluemix.net/[NAMESPACE]/lets-chat
        The push refers to a repository [registry.eu-gb.bluemix.net/[NAMESPACE]/lets-chat] (len: 1)
        Sending image list
        Pushing repository registry.eu-gb.bluemix.net/[NAMESPACE]/lets-chat (1 tags)
        Image adb3157c92fa already pushed, skipping
        Image ed1f86248ba8 already pushed, skipping
        ...
        Image 48b1e23d7a1a already pushed, skipping
        Image 2409eb7b9e8c already pushed, skipping
        Pushing tag for rev [2409eb7b9e8c] on {https://registry.eu-gb.bluemix.net/v1/repositories/ibm_containers_demo_eu/lets-chat/tags/latest}
        
   Now your images are up in the cloud, in your hosted registry, and ready to run on Bluemix.

## Task 2: Verify security vulnerabilities

One of the fundamental aspects of Docker containers is reuse and the ability to base your images on top of other images.  Think of it as inheritance for infrastructure!  But with that comes some heavy responsibility to understand what code you are running on top of and what code you are bringing into your infrastructure through a `docker pull`.  

To solve this issue, IBM Containers provides **Vulnerability Advisor** (VA), a pre-integrated security scanning tool that will alert you of vulnerable images and can even be configured to prevent deployment of those images.  VA can scan any image, regardless of the source, before you deploy a live container from that image. For now, you will look over the vulnerability assessment of the images you just pushed.

1. Go to the [Bluemix Catalog](https://console.eu-gb.bluemix.net/catalog) and click on **Containers**.

  ![catalog](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/7-catalog.jpg)

2. Click on the purple icon for `lets-chat`.  This is the Let's Chat image that you pulled from the public Docker Hub registry, tagged with your namespace, and pushed into your private registry.

  You will see a new page with the vulnerability assessment shown inline.  This is a red/yellow/green scale.  Your Let's Chat image has a yellow status of **Deploy with Caution**.

  ![letschat](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/8-va-lets-chat.jpg)

3. Click on **View report** to see the vulnerability assessment in full detail.

  The *Vulnerable Packages* tab shows you the number of packages scanned, the number of vulnerable packages present in your image, and the number of relevant security notices attached to any of those vulnerable packages.
  You can see that some of the packages are vulnerable.

  ![imagedetail](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/9-va-lets-chat-details.jpg)

  ![imagedetail2](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/10-va-lets-chat-details.jpg)

4. From the vulnerability report page, click the **Back to Catalog** link. As a Bluemix Organization Manager, you can click on **Manage policies**.

  The Manage policies page allows users with the appropriate level of authority to control which images can be deployed based on the vulnerability status.  You can see the multiple options that allows users to *Warn* or *Block* image deployment.

  It also contains a summary view of the state of all images in your registry.  Images can have statuses of *Deployment Blocked*, *Deploy with Caution*, and *Safe to Deploy*.  This gives you a quick look into which images are troublesome and which images are secure across your entire registry.

  > The containers tab can be used to show a similar view showing the state of running containers, rather than images in the registry.

  ![vapolicymgr](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/11-va-policy-mgr-defaults.jpg)

  You will note that the default action is to **Warn** deployments that have vulnerabilities. Click the **Back to Catalog** link.

5. To make the image safe to deploy, we can rebuild the images and update the vulnerable packages

  To do so, create a new directory called `wrapper` at your Terminal window.  
 
        $ mkdir wrapper

  Switch to that directory and run the following commands to create a Dockerfile  

        $ cd wrapper
        $ echo "FROM sdelements/lets-chat:latest" > Dockerfile
        $ echo "USER root" >> Dockerfile
        $ echo "RUN apt-get update && apt-get install -y curl apt openssl tar" >> Dockerfile
        $ echo "USER node" >> Dockerfile


  This will create a new Dockerfile that we can use to build an image.

  Understanding the Dockerfile:  
    - The `FROM` line specifies what image to base our image on.  
    - `USER` is the equivalent of `su` and is required so the build has privileges to run the next command.  
    - `RUN` runs the following command in the container. In this case we are upgrading the vulnerable packages.  
    - Switch the user back to `node`, which is the user we want to run the application as.  

  We are now going to use `cf ic build` to build the image in the cloud and push the image into the registry in one operation.

  **NOTE:** There is a period "." at the end of this docker build command that is required for the command to run.

        cf ic build -t registry.eu-gb.bluemix.net/[NAMESPACE]/lets-chat .


  Understanding this command's components:  
    - The `cf ic build` is standard from the cf ic CLI.  
    - The `registry.eu-gb.bluemix.net` is the fully qualified domain name path to the Registry server  
    - The `NAMESPACE` is each user's unique namespace and identifies your private registry.  
    - The `lets-chat` is the name given to this newly created image.  
    - The `.` specifies that the build command is running using the Dockerfile in this directory.  


6. Go back to the UI and refresh. You will see that the image is **Safe to Deploy**.

  You have reviewed your pushed images, which were sourced from a public repository, and can now deploy them on your hosted Bluemix account.  This is a key step in making sure you are running the code which you expect to be running and you are not opening your organization up to security issues, at the expense of agility.  You still want to stay secure, even when moving at light-speed!

## Task 3: Run your web app on Bluemix

Now that you've pushed your images to Bluemix and reviewed the contents of those images through the IBM Containers Vulnerability Advisor, you can run your images the same way you did locally but without any worry of keeping your laptop on all day, every day!  The commands here you'll run aren't that much different than what you did locally.

1. Back at the Terminal, look and see what images are in your Bluemix hosted registry

        $ cf ic images
        REPOSITORY                                                            TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
        registry.eu-gb.bluemix.net/[NAMESPACE]/lets-chat           latest              3aeb3c224c6b        5 minutes ago       241.5 MB
        registry.eu-gb.bluemix.net/[NAMESPACE]/mongo               latest              ae293c6896a1        6 minutes ago       0 B
        registry.eu-gb.bluemix.net/ibm-node-strong-pm                         latest              ef21e9d1656c        13 days ago         528.7 MB
        registry.eu-gb.bluemix.net/ibmliberty                                 latest              2209a9732f35        13 days ago         492.8 MB
        registry.eu-gb.bluemix.net/ibmnode                                    latest              8f962f6afc9a        13 days ago         429 MB
        registry.eu-gb.bluemix.net/ibm-mobilefirst-starter                    latest              5996bb6e51a1        13 days ago         770.4 MB

2. Now run your MongoDB container just like you did locally, except this time use `cf ic` instead of `docker` to point to Bluemix.      

        $ cf ic run --name lc-mongo -p 27017 -m 128 registry.eu-gb.bluemix.net/[NAMESPACE]/mongo
        71eb28dc-4d95-4a6d-bcaa-93f2382e48b5
 

  Show the running container instances.  If the state of the container is not `RUNNING` re-run the command until the state is `RUNNING` before you proceed:
 
        $ cf ic ps
        CONTAINER ID        IMAGE                                                            COMMAND             CREATED             STATUS                   PORTS               NAMES
        7ebf51a3-35a        registry.eu-gb.bluemix.net/[NAMESPACE]/mongo:latest   ""                  45 seconds ago      Running 27 seconds ago   27017/tcp           lc-mongo


3. Next run your Let's Chat container just like you did locally, again using `cf ic` instead of `docker` to point to Bluemix.

  Run your Let's Chat instance:  

        $ cf ic run --name lets-chat --link lc-mongo:mongo -p 8080 -m 128 registry.eu-gb.bluemix.net/[NAMESPACE]/lets-chat
        a5dc5e0d-8eae-44a2-9f8d-548112bec250


  Show the running container instances.  Wait for a state of `RUNNING` before you proceed:
   

        $ cf ic ps
        CONTAINER ID        IMAGE                                                                COMMAND             CREATED             STATUS                   PORTS               NAMES
        d368a598-69d        registry.eu-gb.bluemix.net/[NAMESPACE]/lets-chat:latest   ""                  10 seconds ago      Building 7 seconds ago   8080/tcp            lets-chat
        7ebf51a3-35a        registry.eu-gb.bluemix.net/[NAMESPACE]/mongo:latest       ""                  2 minutes ago       Running a minute ago     27017/tcp           lc-mongo


4. Finally, you might need to expose your Let's Chat container to the public internet, so you and your team can start chatting!

  Run the `ip list` command to see which IPs are available and then bind one to your running container.

  List available IP addresses for your account:
   

        $ cf ic ip list
        Number of allocated public IP addresses:  0
      
        IpAddress        ContainerId   
    

  If you have no available IP addresses, request one:
   
        $ cf ic ip request
        Successfully requested ip 134.XXX.YYY.ZZZ
 
 
  Once  you have an available IP address, bind that IP to your container:
    
  
        $ cf ic ip bind 134.XXX.YYY.ZZZ lets-chat
        OK
        The IP address was bound successfully
 

  List your containers. Note that your Let's Chat container has an IP address:
   
        $ cf ic ps
        CONTAINER ID        IMAGE                                                                COMMAND             CREATED              STATUS                  PORTS                          NAMES
        d368a598-69d        registry.eu-gb.bluemix.net/[NAMESPACE]/lets-chat:latest   ""                  About a minute ago   Running a minute ago    134.XXX.YYY.ZZ0:8080->8080/tcp   lets-chat
        7ebf51a3-35a        registry.eu-gb.bluemix.net/[NAMESPACE]/mongo:latest       ""                  3 minutes ago        Running 3 minutes ago   27017/tcp                      lc-mongo

5. Check out your running app in your browser, at the IP you just bound.  Remember to use port `8080`!  

![letschat1](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/14-lets-chat.jpg)
![letschat2](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/15-lets-chat.jpg)
![letschat3](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/16-lets-chat.jpg)

## Task 4: Cleanup

To ensure you have enough free quota to continue with the lab, let's clean up your container instances.  This can be done through the UI and the **DELETE** button on each container, or you can do this through the CLI with the following commands.

1. To use the command line:
     
        $ cf ic stop lets-chat
        $ cf ic stop lc-mongo
        $ cf ic rm -f lets-chat
        $ cf ic rm -f lc-mongo


## Congratulations!!!  You have successfully accomplished Lab 2.

#### Let's recap what you've accomplished thus far:

- Tagged our images with our Bluemix namespace.
- Pushed (uploaded) our images to our private registry in Bluemix on the public IBM Cloud.
- Learned about the security posture of our image using Vulnerability Advisor.
- Ran our first containers in the cloud.

### Time to continue with [Lab 3 - Container Group Scaling and Recoverability](3-Container-Group-Scaling-and-Recoverability.md)

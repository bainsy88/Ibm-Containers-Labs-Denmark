# Lab 4: Container Group High Availability

> **Difficulty**: Intermediate

> **Time**: 20 minutes

> **Tasks**:
>- [Prerequisites](#prerequisites)
- [Task 1: Create a Second Bluemix Space in Your Org](#task-1-setup-a-second-bluemix-space-in-your-org)
- [Task 2: Create a Container Group in Each Availability Zone](#task-2-create-a-container-group-in-each-availability-zone)
- [Task 3: Verify Scalability](#task-3-verify-scalability)
- [Task 4: Verify High Availability](#task-4-verify-high-availability)

## Prerequisites

Prior to running this lab, you must have completed the [Prerequisites](https://github.com/bainsy88/containers-denmark/blob/master/0-prereqs.md).

Bluemix trial accounts do not have enough quota resources to complete this tutorial. To complete this tutorial, you must upgrade to a pay as you go Bluemix account.

## Task 1: Set up a Second Bluemix Space in Your Organization
  
1. Create a second Bluemix space.

    - From the Bluemix Dashboard, click **Account** > **Manage Organizations**.
    - Identify the Organization that you want to add a space to, and select **View Details**.
    - Click **Add a Space**.
    - Enter the space name.
    - Click **Add**.

  ![Screenshot indicating the Add a Space button](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/35-add-space.jpg)

2. Ensure that your two spaces are provisioned in different availability zones. In one of your spaces, run the following command:

        $ cf ic info
                                    
        Date/Time                2017-02-22 13:41:05.102777664 +0000 GMT   
        Debug Mode               false   
        Host/URL                 https://containers-api.eu-gb.bluemix.net   
        Registry Host            registry.eu-gb.bluemix.net   
        Bluemix API Host/URL     https://api.eu-gb.bluemix.net   
        Bluemix Org              jack.baines@uk.ibm.com(8a5dc348-6b96-4b35-a6e8-ebe01f7bac25)   
        Bluemix Space            v2(64b24248-d30b-42ad-9122-20d6a259addc)   
        CLI Version              0.8.964   
        API Version              3.0  Wed Feb 15 17:42:02 2017   
        Namespace                bainsy88   
        Availability Zone        lon02-01   
        ...

 
  In the above example, note that this space is provisioned in the `lon02-01` Availability Zone. Next, switch to the second space.


        $ cf target -s [space2_name]
                        
        API endpoint:   https://api.eu-gb.bluemix.net (API version: 2.54.0)
        User:           jack.baines@uk.ibm.com
        Org:            jack.baines@uk.ibm.com
        Space:          [space2_name]


  Now you are targeting the second space, re-run `cf ic info` to check the availability zone.


       $ cf ic info
                                   
       Date/Time                2017-02-22 14:06:18.007770003 +0000 GMT   
       Debug Mode               false   
       Host/URL                 https://containers-api.eu-gb.bluemix.net   
       Registry Host            registry.eu-gb.bluemix.net   
       Bluemix API Host/URL     https://api.eu-gb.bluemix.net   
       Bluemix Org              jack.baines@uk.ibm.com(8a5dc348-6b96-4b35-a6e8-ebe01f7bac25)   
       Bluemix Space            space2(4270de3e-2133-434f-ae74-c53ba6e330a5)   
       CLI Version              0.8.964   
       API Version              3.0  Wed Feb 15 17:20:10 2017   
       Namespace                bainsy88   
       Availability Zone        ams03-01
       

  In the above example, the second space is provisioned in the `ams03-01` Availability Zone. Any containers started in this space run in the Amsterdam datacenter.

  If both spaces are provisioned in the same Availability Zone, run `cf ic reprovision -f [ams03-01 or lon02-01]` to reprovision one of your spaces to the other Availability Zone.

  You now have 2 spaces assigned to different Availability Zones.

## Task 2: Create a Container Group in Each Availability Zone

1. Get your Container namespace for use in subsequent tasks.

        $ export CONTAINER_NAMESPACE=$(cf ic namespace get)

2. Copy the Docker image from Docker Hub to Bluemix:

        $ cf ic cpi ragsns/spring-boot registry.eu-gb.bluemix.net/$CONTAINER_NAMESPACE/spring-boot
        Sending build context to Docker daemon 2.048 kB
        Step 0 : FROM ragsns/spring-boot
         ---> d8dc5ff8d27b
        Successfully built d8dc5ff8d27b
        The push refers to a repository [registry.eu-gb.bluemix.net/ragsns/spring-boot] (len: 1)
        d8dc5ff8d27b: Image already exists 
        3565bce48566: Image already exists 
        bc1666c40f44: Image already exists 
        b6ac4efa6931: Image already exists 
        d7c80008ad18: Image already exists 
        97424a07faef: Image already exists 
        f9194a57b559: Image already exists 
        48a0ef800175: Image already exists 
        3a7cffa50930: Image already exists 
        0c24696b360d: Image already exists 
        696be707a6b0: Image already exists 
        2c00a767497d: Image already exists 
        dc921aeac8d0: Image already exists 
        4c0cc976b7bb: Image already exists 
        1d6f63d023f5: Image already exists 
        ef2704e74ecc: Image already exists 
        Digest: sha256:8a450a05521481b3df8c052f84c1888a7bc1406b0ee2b4ab0146d4dede043c0c

3. Create a Container Group with 3 instances and port 8080 exposed with the following command: 


        $ cf ic group create  -p 8080 --name spring-boot --hostname spring-boot-$CONTAINER_NAMESPACE --domain eu-gb.mybluemix.net --memory 512 --max 3 --desired 3 --auto --anti registry.eu-gb.bluemix.net/$CONTAINER_NAMESPACE/spring-boot
        Create group in progress
        Created group spring-boot (id: 123dbe80-8ae8-434c-ba79-33a64aa82636)
        Minimum container instances: 0
        Maximum container instances: 3
        Desired container instances: 3

  
4. Verify that the group was created with the following command. Eventually the status will show `CREATE_COMPLETE`.


        $ cf ic group list
        Group Id                             Name                                Status                              Created                             Updated                             Port
        123dbe80-8ae8-434c-ba79-33a64aa82636 spring-boot                         CREATE_COMPLETE                     2015-11-20T16:38:33Z                                                    8080

  You now have a container group set up in your second space, running in Amsterdam. We now need to create a second container group in your first space.

5. Target your first space.
  

        $ cf target -s [space1_name]
                        
        API endpoint:   https://api.eu-gb.bluemix.net (API version: 2.54.0)
        User:           jack.baines@uk.ibm.com
        Org:            jack.baines@uk.ibm.com
        Space:          [space1_name]


6. Repeat steps 1 - 4 to create another container group in the first space. This container group uses the same external route, but runs in the other Availability Zone.

## Task 3: Verify Scalability

The `spring-boot` container has a web service endpoint `/env` that we will invoke now as below.

1. List all the environment variables associated with the application using the following command:

        $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env
        Environment : 
        tenant_id = 49911e9c-cf3e-4913-9e20-ff52ed6d34e3
        PATH = /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
        DOCKER_WAITFORNET = true
        sgroup_id = 241262d9-90a4-4764-bf0a-c48ef8e13b36
        logstash_target = logmet.opvis.bluemix.net:9091
        logging_password = 
        tagformat = tenant_id group_id uuid
        CA_CERTIFICATES_JAVA_VERSION = 20140324
        TERM = xterm
        LANG = C.UTF-8
        uuid = 66ad3b7c-de2d-4707-a7a8-f894302e52e7
        metrics_target = logmet.opvis.bluemix.net:9095
        sgroup_name = spring-boot
        HOSTNAME = instance-0010b64e
        JAVA_DEBIAN_VERSION = 8u66-b17-1~bpo8+1
        group_id = 241262d9-90a4-4764-bf0a-c48ef8e13b36
        tagseparator = _
        PWD = /
        JAVA_VERSION = 8u66
        space_id = 49911e9c-cf3e-4913-9e20-ff52ed6d34e3
        HOME = /root

2. We are primarily interested in the `HOSTNAME`. A container group automatically creates a load balancer which distributes requests amongst these different instances. If you invoke the command repeatedly you will see as many different instances as the size of the container group and no more (in this case three).

If you invoke the command repeatedly, you will see as many different instances IDs as the total number of instances across both groups (in this case six).

        $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
        HOSTNAME = instance-000abaec
      
        $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
        HOSTNAME = instance-00122559
      
        $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
        HOSTNAME = instance-010eef7c
      
        $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
        HOSTNAME = instance-004369e6
      
        $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
        HOSTNAME = instance--004369e9  
      
        $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
        HOSTNAME = instance-004369e3
 
You will start to see the same instances recycle after a while depending on how the load balancer balances the load.
        
        $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
        HOSTNAME = instance-00122559


## Task 4: Verify High Availability

1. Verify that the group in the current space is still active.


        $ cf ic group list
        Group Id                             Name                                Status                              Created                             Updated                             Port


2. We will remove the group that is running in the currently targeted space. This is to simulate an outage in the Amsterdam Datacenter. 


        $ cf ic group rm -f spring-boot



3. Invoking the command immediately after deleting will show the group is in `DELETE_IN_PROGRESS` state.
        
        $ cf ic group list
        Group ID                               Name          Status               Created                         Updated   Port   
        0254ec35-6fb5-4514-9aca-bef3fcdc3215   spring-boot   DELETE_IN_PROGRESS   2017-02-22 14:29:01 +0000 GMT             0                                                          8080

  
4. Curl the route again and verify it is still responding.

 
        $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/
        Hello World!


  The request is still completed seccessfully because it is routed to the container group in London.

5. Switch to your other space so we can validate the group is still running:


        $ cf target -s [space1_name]
                        
        API endpoint:   https://api.eu-gb.bluemix.net (API version: 2.54.0)
        User:           jack.baines@uk.ibm.com
        Org:            jack.baines@uk.ibm.com
        Space:          [space1_name]
        
        
        $ cf ic group list
          Group ID                               Name          Status            Created                         Updated                         Port   
        76f0178b-74a4-48eb-bd13-4370897e6050   spring-boot   UPDATE_COMPLETE   2017-02-22 11:34:01 +0000 GMT   2017-02-22 11:44:02 +0000 GMT   808

## Cleanup

To continue with another lab, you need to clean up your container group.  This can be done through the UI and the **DELETE** button on each container, or you can do this through the CLI with the following command


        $ cf ic group rm -f spring-boot


## Congratulations!!!  You have successfully accomplished Lab 4.

#### Let's recap what you've accomplished thus far in this lab:

- Created a Second Bluemix Space in Your Org.
- Created a Container Group in Each Availability Zone.
- Verified high availability by deleting one of the two container groups and demonstrated that requests are still served correctly.

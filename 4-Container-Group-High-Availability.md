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

Prior to running this lab, you must have completed the pre-reqs. This lab will also only work for users not on a trial account, due to quota restrictions

## Task 1: Setup a Second Bluemix Space in Your Org
  
1. Create a second Bluemix space.

    - Click Account > Manage Organizations page.
    - Identify the org that you want to add a space to, and select View Details.
    - Click Add a Space.
    - Enter the space name.
    - Click Add.

  ![addspace](https://github.com/bainsy88/containers-denmark/blob/master/screenshots/35-add-space.jpg)

2. Now we have a second space we need to make sure they are assigned to seperate availability zones.

  ```
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
  ```
 
  In the above example you can see that the first space is in zone lon02-01. We now need to target the second space.

  ```
  $ cf target -s [space2_name]
                
API endpoint:   https://api.eu-gb.bluemix.net (API version: 2.54.0)
User:           jack.baines@uk.ibm.com
Org:            jack.baines@uk.ibm.com
Space:          [space2_name]
  ```

  Now we are targeting the second space we can re-run `cf ic info`.

  ```
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
```

As you can see above the second space is in the ams03-01 availabiltity zone. This means that any contianers started in this space will be running in the Amsterdam datacentre.

If the Availability Zone on the second space is the same as first space you can run `cf ic reprovision -f [ams03-01 or lon02-01]` to reprovision the second space to the other Availability Zone.

We now have 2 spaces assigned to different availability zones.

## Task 2: Create a Container Group in Each Availability Zone

1. Use the Container namespace in subsequent tasks.

  ```
	$ export CONTAINER_NAMESPACE=$(cf ic namespace get)
  ```

2. Upload the Docker image from the Docker public registry to Bluemix as we did earlier using the following command.

  ```
	$ cf ic cpi ragsns/spring-boot registry.eu-gb.bluemix.net/$CONTAINER_NAMESPACE/spring-boot
Sending build context to Docker daemon 2.048 kB
Step 0 : FROM bainsy88/spring-boot
 ---> d8dc5ff8d27b
Successfully built d8dc5ff8d27b
The push refers to a repository [registry.eu-gb.bluemix.net/bainsy88/spring-boot] (len: 1)
d8dc5ff8d27b: Image already exists 
3565bce48566: Image already exists 
bc1666c40f44: Image already exists 
b6ac4efa6931: Image already exists 
d7c80008ad18: Image already exists 
de4a13c84f53: Image already exists 
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
  ```  

3. Create a Container Recovery Group with 3 (at least greater than 1) instances and port 8080 exposed with the following command. 

  ```
  $ cf ic group create  -p 8080 --name spring-boot --hostname spring-boot-$CONTAINER_NAMESPACE --domain eu-gb.mybluemix.net --memory 512 --max 3 --desired 3 --auto --anti registry.eu-gb.bluemix.net/$CONTAINER_NAMESPACE/spring-boot
  Create group in progress
Created group spring-boot (id: 0254ec35-6fb5-4514-9aca-bef3fcdc3215)
Minimum container instances: 0
Maximum container instances: 3
Desired container instances: 3
  ```
  
4. Verify the group was created with the following command. Eventually the status will show `CREATE_COMPLETE`.

  ```
  $ cf ic group list
  Group Id                             Name                                Status                              Created                             Updated                             Port
0254ec35-6fb5-4514-9aca-bef3fcdc3215 spring-boot                         CREATE_COMPLETE                     2017-02-22 14:29:01 +0000 GMT                                                    8080
  ```

You now have a container group set up in space 2, running in Amsterdam. We now need to create a second container group in the first space.

5. Target the first space.
  
  ```
  $ cf target -s [space1_name]
                
API endpoint:   https://api.eu-gb.bluemix.net (API version: 2.54.0)
User:           jack.baines@uk.ibm.com
Org:            jack.baines@uk.ibm.com
Space:          [space1_name]
  ```

6. Repeat steps 1 - 4 now you are targetting the first space. This will create another container group using the same external route, but this time running in London.
  
## Task 3: Verify Scalability

There is a web service endpoint `/env` that we will invoke now as below.

1. List all the environment variables associated with the application using the following command.

  ```
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
  ```

2. We are primarily interested in the `HOSTNAME`. A container recovery group automatically creates a load balancer and load balances amongst these different instances. In additon to the load balancing within the group, we are also going to load balance across both the groups, running in different datacentres, as they are both bound to the same route.

If you invoke the command repeatedly you will see as many different instances id's as the total number of instances across both groups(in this case six).

  ```
  $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
  HOSTNAME = instance-000abaec
  ```

  ```
  $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
  HOSTNAME = instance-00122559
  ```
  
  ```
  $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
  HOSTNAME = instance-010eef7c
  ```
  
  ```
  $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
  HOSTNAME = instance-004369e6
  ```

  ```
  $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
  HOSTNAME = instance--004369e9
  ```
  
  ```
  $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
  HOSTNAME = instance-004369e3
  ```
You will start to see the same instances recycle after a while depending on how the load balancer balances the load.

  ```
  $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/env | grep HOSTNAME
  HOSTNAME = instance-00122559
  ```

## Task 4: Verify High Availability

1. Verify that the group in the current space is still active.

  ```
  $ cf ic group list
  Group Id                             Name                                Status                              Created                             Updated                             Port
0254ec35-6fb5-4514-9aca-bef3fcdc3215 spring-boot                         CREATE_COMPLETE                     2017-02-22 14:29:01 +0000 GMT                                                    8080
  ```

2. We will remove the group that is running in the currently targetted space. This is to simulate an outage in the Amsterdam Datacentre 

  ```
$ cf ic group rm -f spring-boot
  ```


3. Invoking the command immediately after deleting will show the group is in DELETE_IN_PROGRESS state.

  ```
$ cf ic group list
Group ID                               Name          Status               Created                         Updated   Port   
0254ec35-6fb5-4514-9aca-bef3fcdc3215   spring-boot   DELETE_IN_PROGRESS   2017-02-22 14:29:01 +0000 GMT             0                                                          8080
  ```
  
4. Curl the route again and verify it is still responding.

  ```
  $ curl -L spring-boot-$CONTAINER_NAMESPACE.eu-gb.mybluemix.net/
Hello World!
  ```

  The reason this still works is the requests are being routed to the container group that is still running in space 1 which is provisioned in London

5. Switch to the your other space so we can validate the group is still running in space 1 

  ```
  $ cf target -s [space1_name]
                
API endpoint:   https://api.eu-gb.bluemix.net (API version: 2.54.0)
User:           jack.baines@uk.ibm.com
Org:            jack.baines@uk.ibm.com
Space:          [space1_name]
  ```

  ```
  Group ID                               Name          Status            Created                         Updated                         Port   
76f0178b-74a4-48eb-bd13-4370897e6050   spring-boot   UPDATE_COMPLETE   2017-02-22 11:34:01 +0000 GMT   2017-02-22 11:44:02 +0000 GMT   808
  ```
## Cleanup

To continue with another lab, you need to clean up your container group.  This can be done through the UI and the **DELETE** button on each container, or you can do this through the CLI with the following command

  ```
$ cf ic group rm -f spring-boot
  ```

##Congratulations!!!  You have successfully accomplished Lab 4.

####Let's recap what you've accomplished thus far in this lab:

- Created a Second Bluemix Space in Your Org]
- Created a Container Group in Each Availability Zone
- Verified high availability by deleting the group in space 2 and confirming we could still serve requests

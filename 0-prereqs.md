### IBM Containers - solving problems at light speed with a hosted Docker service
##### Prerequisite information necessary to prepare for containers lab
###### Hands-On Lab Demonstrating the Enterprise-Grade Capabilities of IBM Containers

1. Pre-requisite installation
    * Install Docker 1.8.1 or later from [Docker](https://www.docker.com/products/docker).  
      - Note if you are using Windows 7 or below you will need to use [Docker Toolbox](https://www.docker.com/docker-toolbox) instead. When you are instructed to open a Terminal window, run Docker Quickstart Terminal from your Start menu.
    * Install the [Cloud Foundry CLI version 6.12.0 or later][cloud-foundry-cli] from the GitHub repository
    * Install the [IBM Containers plugin][ibm-containers-cli] for the Cloud Foundry CLI
    
    
    OS X
    cf install-plugin https://static-ice.ng.bluemix.net/ibm-containers-mac

    Linux 64-bit
    cf install-plugin https://static-ice.ng.bluemix.net/ibm-containers-linux_x64
    
    Linux 32-bit
    cf install-plugin https://static-ice.ng.bluemix.net/ibm-containers-linux_x86

    Windows 64-bit
    cf install-plugin https://static-ice.ng.bluemix.net/ibm-containers-windows_x64.exe

    Windows 32-bit
    cf install-plugin https://static-ice.ng.bluemix.net/ibm-containers-windows_x86.exe

When the installation completes, an OK message is displayed. 
2. Sign up for an IBM Bluemix account at [Bluemix Signup][bluemix-signup-link]
   ![Bluemix](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/1-bluemix-signup.jpg)
   ![Signup](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/4-bluemix-trial.jpg)
3. Log into IBM Bluemix
     ![Bluemix](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/2-bluemix-login.jpg)
     ![Bluemix](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/3-bluemix-login.jpg)
4. Follow instructions to create an org and space
     ![Bluemix](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/31-bluemix-wizard.jpg)
     ![Bluemix](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/32-bluemix-wizard.jpg)
     ![Bluemix](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/33-bluemix-wizard.jpg)
     ![Bluemix](https://github.com/bainsy88/containers-denmark/raw/master/screenshots/34-bluemix-wizard.jpg)

You are now ready to begin [Lab 1 - Introduction to IBM Containers and Docker](1-Intro-to-IBM-Containers-and-Docker.md).


[bluemix-signup-link]: https://console.eu-gb.bluemix.net
[cloud-foundry-cli]: https://github.com/cloudfoundry/cli/releases
[ibm-containers-cli]: https://console.ng.bluemix.net/docs/containers/container_cli_cfic_install.html

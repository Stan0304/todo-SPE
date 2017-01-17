![](./images/container_icon64x64.png)
# Introduction

In this lab, you’ll gain a high level understanding of the architecture, features, and development concepts related to the IBM Containers (IC) service. Throughout the lab, you’ll get a chance to use the Command Line Interface (CLI) for deploying new docker images to Bluemix, manage your running container, and use the API. This lab'll use the open-source [NGINX](https://www.nginx.com/) docker image.


# Objective

In the following lab, you will learn:

+ How to pull a docker image from docker hub and push it to Bluemix
+ How to bind a routable IP address to a running container
+ How to run a single container and a group of containers on Bluemix
+ How to use the IBM Containers API


# Pre-Requisites

+ Get a [Bluemix IBM id](https://bluemix.net)
+ Install docker for [Mac](https://docs.docker.com/engine/installation/mac/) or [Windows](https://docs.docker.com/engine/installation/windows/)
+ Install the [Cloud Foundry Command-Line CLI](https://github.com/cloudfoundry/cli/releases)


# Steps

1. [Start an existing docker image on Bluemix](#step-1---start-an-existing-docker-image-on-bluemix)
2. [Pull and run a container locally](#step-2---pull-and-run-a-container-locally)
3. [Prepare your IBM Containers service](#step-3---prepare-your-ibm-containers-service)
4. [Push and run your image on Bluemix](#step-4---push-and-run-your-image-on-bluemix)
5. [Attach an IP to your container](#step-5---attach-an-ip-to-your-container)
6. [Create a highly available group](#step-6---create-a-highly-available-group)
7. [Use the Container API](#step-7---use-the-container-api)


# Step 1 - Start an existing docker image on Bluemix

In this first step, you will learn how to use the Bluemix Console to manage container following the documentation [Getting started with IBM Containers](https://console.ng.bluemix.net/docs/containers/container_index.html).

You could skip this step and directly start Step 2 if you only wanted to interact with the command line.


# Step 2 - Pull and run a container locally

1. Search the NGINX official image from Docker Hub
  ```
  docker search nginx
  ```

1. Retrieve the image NGINX locally on your laptop
  ```
  docker pull nginx
  ```

1. Let's start our NGINX Docker container with this command:
  ```
  docker run -d -p 80:80 --name webserver nginx
  ```
  
  + ```run``` is the command to create a new container
  + The ```-d``` flag to run this container in the background. The container will run in detached mode, meaning the container is started and stays running until stopped but does not listen to the command line.
  + ```-p``` specifies the port we are exposing in the format of -p local-machine-port:internal-container-port. The NGINX image exposes ports 80 and 443 in the container. In this case, we are mapping the container port 80 to port 80 on the Docker host.
  + ```--name``` flag is how we specify the name of the container (if left blank one is assigned for us)
  + ```nginx``` is the name of the image on dockerhub (we downloaded this before with the pull command, but Docker will do this automatically if the image is missing)

1. Run ```docker ps``` to verify that the container was created and is running

1. Show the running server: [http://localhost:80](http://localhost:80); the default NGINX welcome page appears.

  Note: If you have installed Docker Toolbox, you need to retrieve the IP address of the Virtual Box VM by running the following command:

  ```docker-machine ip default```

  Then show the running server with [http://IP_ADDRESS_OF_DOCKER_MACHINE:80](http://IP_ADDRESS_OF_DOCKER_MACHINE:80)


# Step 3 - Prepare your IBM Containers service

To run native Docker CLI commands to manage your containers, we will use the ```cf ic```command line, which is an extension to the Cloud Foundry CLI.

1. Open a command line utility.

1. To install cf CLI plug-ins from the Bluemix registry, set the plug-in registry endpoint:
  
  ```
  cf add-plugin-repo bluemix-cf https://plugins.ng.bluemix.net
  ```

1. Run the following command to install a plug-in:
  
  ```
  cf install-plugin IBM-Containers -r bluemix-cf
  ```

1. Connect to Bluemix

  ```
  $ cf api <Bluemix_endpoint>
  ```

  Select one of the Bluemix endpoint below based on the region where you created your app.
  * US: https://api.ng.bluemix.net
  * EU: https://api.eu-gb.bluemix.net
  * AU: https://api.au-syd.bluemix.net
  
1. Login to Bluemix

  ```
  $ cf login
  ```
  
1. Authenticate to the IBM Containers registry
  
  ```
  cf ic init
  ```

1. Check that you’re connected
  
  ```
  cf ic info
  ```

1. Set the namespace for your organization if this is the first time you log in to your Container Private Registry. A namespace is a unique name to identify your private Docker images registry in Bluemix. When you create a container, you must specify an image's location by including the namespace with the image name.

  ```
  cf ic namespace set <NEW_NAMESPACE>
  ```
  
  Note: The namespace is assigned one time for an organization and cannot be changed after it is created. If this organization namespace was already assigned, you can run the command below to retrieve it:
  
  ```
  cf ic namespace get
  ```


# Step 4 - Push and run your image on Bluemix
  
1. Tag image for Bluemix registry
  ```
  docker tag nginx:latest registry.eu-gb.bluemix.net/<YOUR_NAMESPACE>/nginx:latest
  ```
 
1. Push ​*NGINX*​ image to Bluemix Public
  ```
  docker push registry.eu-gb.bluemix.net/<YOUR_NAMESPACE>/nginx:latest
  ```

1. Validate the presence of ​*NGINX*​ image on Bluemix
  ```
  cf ic images
  ```

1. Start the NGINX image on Bluemix
  ```
  cf ic run -d -p 80 --name webserver registry.eu-gb.bluemix.net/<YOUR_NAMESPACE>/nginx:latest
  ```

  Note: As each container has its own IP, there is no risk of port conflict. Thus, port mapping is not required.
  
  Note: In case you don't need to run the image locally, you could copy directly the docker image from Docker Hub to your Bluemix private registry by running the command below. The image will be automatically tag.
  
  ```cf ic cpi nginx nginx```


# Step 5 - Attach an IP to your container

1. List running containers on Bluemix. Write down the ID of the running NGINX container.
  ```
  cf ic ps
  ```
  
1. Reqest a routable IP address.
  ```
  cf ic ip request
  ```

1. Bind this IP address with your container
  ```
  cf ic ip bind <IP_ADDRESS> <YOUR_NGINX_CONTAINER_ID>
  ```

1. Show the running container on Bluemix: http://YOUR_IP_ADDRESS:80


# Step 6 - Create a highly available group

1. Create a highly available group with 3 containers deployed with anti-affinity and auto-recovery and accessed via a public URL.

  ```
  cf ic group create --name nginx-mygroup --desired 3 -n nginx-mynode -d eu-gb.mybluemix.net -p 80 --anti --auto registry.eu-gb.bluemix.net/mace/nginx:latest
  ```

1. Once the group is started, you can access it:

  ```
  URL: http://my-nginx-node.eu-gb.mybluemix.net
  ```
 
1. You can create a highly available container group that is accessed via a public IP if you would like to control SSL termination yourself and you don't require IBM's edge service for public routing. Shown below reusing the same public IP address from your initial single container in Step 5 above.

  ```
  cf ic group create --name nginx-mygroup-ip --desired 3 -ip <IP_ADDRESS> -p 80 --anti --auto registry.eu-gb.bluemix.net/mace/nginx
  ```
 
1. Once the group is started, you can access it:

  ```
  URL:  http://<IP_ADDRESS>
  ```
 
1. You can list your groups using.
 
  ```
  cf ic group list
  ```

  Note: If your group ends up with a state equals to NETWORK_DEGRADED, this is most likely due to the port you supply to the `cf ic group create` command doesn’t respond with a http 200. Meaning the container is listening on one port, but the group was told a different port.

# Step 7 - Use the Container API

1. Containers API is available in [Swagger Container API](http://ccsapi-doc.mybluemix.net) 

1. This API requires two HTTP headers. To retrieve those values, run the commands next to each header:

  X-Auth-Token     = ```cf oauth-token```
  
  X-Auth-Project-Id= ```cf space <SPACE_NAME> --guid```

1. To retrieve a namespace, run the following command with the correct HTTP headers instead of XXXXX
  ```
  curl -X GET -H "X-Auth-Project-Id: XXXXX" -H "Accept: application/json" -H "X-Auth-Token: bearer XXXXX" "https://containers-api.eu-gb.bluemix.net/v3/registry/namespaces"
  ```


# Resources

For additional resources pay close attention to the following:

- [Getting started with IBM Containers](https://console.ng.bluemix.net/docs/containers/container_index.html)
- [Running highly available processes as container groups](https://console.eu-gb.bluemix.net/docs/containers/container_ha.html)
- [See Auto-scaling in Action for IBM Containers - YouTube](https://www.youtube.com/watch?v=MFs-pSr2gsw)
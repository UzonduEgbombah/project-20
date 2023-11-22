# Migration to the Сloud with containerization. Part 1 - Docker & Docker Compose

Until now, you have been using VMs (AWS EC2) in Amazon Virtual Private Cloud (AWS VPC) to deploy your web solutions, and it works well in many cases. You have learned how easy to spin up and configure a new EC2 manually or with such tools as Terraform and Ansible to automate provisioning and configuration. You have also deployed two different websites on the same VM; this approach is scalable, but to some extent; imagine what if you need to deploy many small applications (it can be web front-end, web-backend, processing jobs, monitoring, logging solutions, etc.) and some of the applications will require various OS and runtimes of different versions and conflicting dependencies - in such case you would need to spin up serves for each group of applications with the exact OS/runtime/dependencies requirements. When it scales out to tens/hundreds and even thousands of applications (e.g., when we talk of microservice architecture), this approach becomes very tedious and challenging to maintain.
In this project, we will learn how to solve this problem and begin to practice the technology that revolutionized application distribution and deployment back in 2013! We are talking of Containers and imply Docker. Even though there are other application containerization technologies, Docker is the standard and the default choice for shipping your app in a container!

#### Install Docker and prepare for migration to the Cloud
First, we need to install Docker Engine, which is a client-server application that contains:

- A server with a long-running daemon process dockerd.
- APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
- A command-line interface (CLI) client docker.

  ![](https://github.com/UzonduEgbombah/project-20/assets/137091610/cb545c19-3b3e-4c7c-bb74-7a7f334ef861)

Before we proceed further, let us understand why we even need to move from VM to Docker.
As you have already learned - unlike a VM, Docker allocated not the whole guest OS for your application, but only isolated minimal part of it - this isolated container has all that your application needs and at the same time is lighter, faster, and can be shipped as a Docker image to multiple physical or virtual environments, as long as this environment can run Docker engine. This approach also solves the environment incompatibility issue. It is a well-known problem when a developer sends his application to you, you try to deploy it, deployment fails, and the developer replies, "- It works on my machine!". With Docker - if the application is shipped as a container, it has its own environment isolated from the rest of the world, and it will always work the same way on any server that has Docker engine.

To begin our migration project from VM based workload, we need to implement a Proof of Concept (POC). In many cases, it is good to start with a small-scale project with minimal functionality to prove that technology can fulfill specific requirements. So, this project will be a precursor before you can move on to deploy enterprise-grade microservice solutions with Docker. And so, Project 21 through to 30 will gradually introduce concepts and technologies as we move from POC onto enterprise level deployments.
You can start with your own workstation or spin up an EC2 instance to install Docker engine that will host your Docker containers.
Remember our Tooling website? It is a PHP-based web solution backed by a MySQL database - all technologies you are already familiar with and which you shall be comfortable using by now.
So, let us migrate the Tooling Web Application from a VM-based solution into a containerized one.

#### MySQL in container

Let us start assembling our application from the Database layer - we will use a pre-built MySQL database container, configure it, and make sure it is ready to receive requests from our PHP application.

#### Step 1:
Pull MySQL Docker Image from Docker Hub Registry

Start by pulling the appropriate Docker image for MySQL. You can download a specific version or opt for the latest release, as seen in the following command:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/13ab1cba-2f27-485e-9337-a506c8f447d1)

If you are interested in a particular version of MySQL, replace latest with the version number. Visit Docker Hub to check other tags here
List the images to check that you have downloaded them successfully:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/7d56bc60-7656-4060-92f3-e7362863fc04)

#### Step 2:
Deploy the MySQL Container to your Docker Engine
Once you have the image, move on to deploying a new MySQL container with:

docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql/mysql-server:latest

- Replace <container_name> with the name of your choice. If you do not provide a name, Docker will generate a random one
- The -d option instructs Docker to run the container as a service in the background
- Replace <my-secret-pw> with your chosen password
In the command above, we used the latest version tag. This tag may differ according to the image you downloaded


Then, check to see if the MySQL container is running: Assuming the container name specified is mysql-server

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/8cf1f412-53df-4deb-98c8-2353f02d956e)

You should see the newly created container listed in the output. It includes container details, one being the status of this virtual environment. The status changes from health: starting to healthy, once the setup is complete.

#### Step 3:
Connecting to the MySQL Docker Container
We can either connect directly to the container running the MySQL server or use a second container as a MySQL client. Let us see what the first option looks like.
#### Approach 1
Connecting directly to the container running the MySQL server:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/3a2eab96-a223-49ef-82da-1200795c9900)

Provide the root password when prompted. With that, you have connected the MySQL client to the server.
Finally, change the server root password to protect your database.
# Approach 2
First, create a network:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/bf4537b0-1ef0-48ed-b15a-fe11822cb475)

Creating a custom network is not necessary because even if we do not create a network, Docker will use the default network for all the containers you run. By default, the network we created above is of DRIVER Bridge. So, also, it is the default network. You can verify this by running the docker network ls command.
But there are use cases where this is necessary. For example, if there is a requirement to control the cidr range of the containers running the entire application stack. This will be an ideal situation to create a network and specify the --subnet
For clarity's sake, we will create a network with a subnet dedicated for our project and use it for both MySQL and the application so that they can connect.
Run the MySQL Server container using the created network.
First, let us create an environment variable to store the root password:

export MYSQL_PW=<root-secret-password>

Then, pull the image and run the container, all in one command like below:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/98e1bc24-7520-446e-8316-6e74d34063b3)

Flags used


- -d runs the container in detached mode

- --network connects a container to a network

- -h specifies a hostname

If the image is not found locally, it will be downloaded from the registry.
Verify the container is running:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/09b0b3e2-8ede-4d1d-807f-935c15c5e017)

As you already know, it is best practice not to connect to the MySQL server remotely using the root user. Therefore, we will create an SQL script that will create a user we can use to connect remotely.
#### Create a file and name it create_user.sql and add the below code in the file:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/fc0900a5-06b9-4045-90c9-f891831687e5)

Run the script:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/568fc5d9-197d-4fb5-bd29-a9b7c49688ca)
If you see a warning like below, it is acceptable to ignore:

#### Connecting to the MySQL server from a second container running the MySQL client utility
The good thing about this approach is that you do not have to install any client tool on your laptop, and you do not need to connect directly to the container running the MySQL server.
Run the MySQL Client Container:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/03e3cd4a-e2a6-40cd-a615-e01af524686f)

Flags used:


- --name gives the container a name

- -it runs in interactive mode and Allocate a pseudo-TTY

- --rm automatically removes the container when it exits

- --network connects a container to a network

- -h a MySQL flag specifying the MySQL server Container hostname

- -u user created from the SQL script

- -p password specified for the user created from the SQL script


#### Prepare database schema
Now you need to prepare a database schema so that the Tooling application can connect to it.

Clone the Tooling-app repository from here
git clone https://github.com/darey-devops/tooling.git

On your terminal, export the location of the SQL file

You can find the tooling_db_schema.sql in the html folder of cloned repo.

Use the SQL script to create the database and prepare the schema. With the docker exec command, you can execute a command in a running container.

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/cdc2a7c2-fff6-4624-84f5-791844786e78)

Update the db_conn.php file with connection details to the database

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/a9faf76f-50e2-4a6a-81f2-b0b06dee7fa1)

#### Run the Tooling App

Containerization of an application starts with creation of a file with a special name - 'Dockerfile' (without any extensions). This can be considered as a 'recipe' or 'instruction' that tells Docker how to pack your application into a container. In this project, you will build your container from a pre-created Dockerfile, but as a DevOps, you must also be able to write Dockerfiles.
You can watch this video to get an idea how to create your Dockerfile and build a container from it.
And on this page, you can find official Docker best practices for writing Dockerfiles.
So, let us containerize our Tooling application; here is the plan:

Make sure you have checked out your Tooling repo to your machine with Docker engine
First, we need to build the Docker image the tooling app will use. The Tooling repo you cloned above has a Dockerfile for this purpose. Explore it and make sure you understand the code inside it.
Run docker build command
Launch the container with docker run

Try to access your application via port exposed from a container

Let us begin:
Ensure you are inside the folder that has the Dockerfile and build your container:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/54e9bc8f-f65c-441f-96e0-73871c0ff2d8)

In the above command, we specify a parameter -t, so that the image can be tagged tooling"0.0.1 - Also, you have to notice the . at the end. This is important as that tells Docker to locate the Dockerfile in the current directory you are running the command. Otherwise, you would need to specify the absolute path to the Dockerfile.

Run the container:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/074f6428-81a6-4020-bd03-4e2b1515e907)

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/c6ac96ec-e6b7-48f0-9530-bd5e618b5ce9)

Let us observe those flags in the command.

- We need to specify the --network flag so that both the Tooling app and the database can easily connect on the same virtual network we created earlier.
- The -p flag is used to map the container port with the host port. Within the container, apache is the webserver running and, by default, it listens on port 80. You can confirm this with the CMD ["start-apache"] section of the Dockerfile. But we cannot directly use port 80 on our host machine because it is already in use. The workaround is to use another port that is not used by the host machine. In our case, port 8085 is free, so we can map that to port 80 running in the container.

If everything works, you can open the browser and type http://localhost:8085
You will see the login page.

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/60e504b2-9742-468b-88c7-c817b5c9127f)

# Deployment with Docker Compose

All we have done until now required quite a lot of effort to create an image and launch an application inside it. We should not have to always run Docker commands on the terminal to get our applications up and running. There are solutions that make it easy to write declarative code in YAML, and get all the applications and dependencies up and running with minimal effort by launching a single command.
In this section, we will refactor the Tooling app POC so that we can leverage the power of Docker Compose.

- First, install Docker Compose on your workstation from here

- Create a file, name it tooling.yaml

Begin to write the Docker Compose definitions with YAML syntax. The YAML file is used for defining services, networks, and volumes:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/b7207bca-7acd-4e0b-8613-ddc7f84252c5)


The YAML file has declarative fields, and it is vital to understand what they are used for.


version: Is used to specify the version of Docker Compose API that the Docker Compose engine will connect to. This field is optional from docker compose version v1.27.0. You can verify your installed version with:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/12cfa407-c5bf-4a64-b8f9-605430ffdd4e)

service: A service definition contains a configuration that is applied to each container started for that service. In the snippet above, the only service listed there is tooling_frontend. So, every other field under the tooling_frontend service will execute some commands that relate only to that service. Therefore, all the below-listed fields relate to the tooling_frontend service.
- build
- port
- volumes
- links

You can visit the site here to find all the fields and read about each one that currently matters to you -> https://www.balena.io/docs/reference/supervisor/docker-compose/
You may also go directly to the official documentation site to read about each field here -> https://docs.docker.com/compose/compose-file/compose-file-v3/
Let us fill up the entire file and test our application:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/d8c25d98-b4df-4802-be41-556d7a6d9633)

Run the command to start the containers

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/06ba8232-bd88-43bf-97df-4c19117980ad)

Verify that the compose is in the running status:

![](https://github.com/UzonduEgbombah/project-20/assets/137091610/bb44043d-a6d3-4daf-b24b-89ee42b6b383)






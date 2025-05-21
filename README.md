## TP1 Docker

### 1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?

This way you don't write information like the password in the Dockerfile allowing for better security. And also writting the variables in the Dockerfile doesn't allow to create different users for example so we have less flexibility.

### 1-2 Why do we need a volume to be attached to our postgres container?

A volume allows to save locally the data from the container. This can be usefull if the container needs to be rebuilt for any reason without losing its content.

### 1-3 Document your database container essentials: commands and Dockerfile.

#### Dockerfile
```
FROM postgres:17.2-alpine


COPY CreateScheme.sql /docker-entrypoint-initdb.d/
COPY InsertData.sql /docker-entrypoint-initdb.d/
```
#### Then for the commands:
- Create the network
```
docker network create app-network
```
- Create the image from the Dockerfile
```
docker build -t my-postgres-image .
```
- Create the Postgres container with a volume
```
docker run -d --name my-postgres-container --network app-network -e POSTGRES_DB=mydb -e POSTGRES_USER=myuser -e POSTGRES_PASSWORD=mypassword -v "C:\Users\dbouq\OneDrive - Fondation EPF\4A\DevOps\DevOps\TP\TP1\Postgres\data:/var/lib/postgresql/data" my-postgres-image
```
- Create the Adminer container
```
docker run -p "8090:8080" --net=app-network --name=adminer  -d adminer
```
 
### 1-4 Why do we need a multistage build? And explain each step of this dockerfile.

A multistage build allows to only keep what is needed to run the app. For example once the code is compiled, there is no need to keep the tools of the JDK in the image produced by a Dockerfile.

#### Dockerfile
```
FROM eclipse-temurin:21-jdk-alpine AS myapp-build
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME

RUN apk add --no-cache maven

COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run stage
FROM eclipse-temurin:21-jre-alpine
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT ["java", "-jar", "myapp.jar"]
```

The first part is used to import tools like jdk21 and Maven to compile the code and create a .jar with the compiled code:
```
FROM eclipse-temurin:21-jdk-alpine AS myapp-build
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME

RUN apk add --no-cache maven

COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests
```
Then in the second part a JRE is imported that only has the minimum requierments to run the app. Then, only the .jar and JRE are copied to the container.
```
# Run stage
FROM eclipse-temurin:21-jre-alpine
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT ["java", "-jar", "myapp.jar"]
```

### 1-5 Why do we need a reverse proxy?

A reverse proxy is used to do the link with the backend usaly in a more user-friendly. It is also separed from the database making it more secured.

### 1-6 Why is docker-compose so important?

A docker compose file allows to manage easily application with different containers, networks, configurations in one file. It also makes it easier and more consitent to run an the containers of an app.

### 1-7 Document docker-compose most important commands.

```
docker-compose up
```
It builds, creates or recreates, starts and connects the different containers.

```
docker-compose up --build	
```
Same as previous but it also forces the rebuilt of the images before running the containers

```
docker-compose down	
```
Stop and removes containers and networks

```
docker-compose down	-v
```
Same but also removes the volumes.

```
docker-compose logs	
```
Shows the logs of the different containers

```
docker-compose ps
```
Shows the different containers

### 1-8 Document your docker-compose file.

```
services:
  database:
    build:

    # Build an image called database from the Dockerfile in the context directory
      context: ./Postgres
    image: database

    # Indicates the .env file containing the different variables needed to connect to the database
    env_file:
      - .env

    # Indicates the path of the folder where the data from the container will be stored in a persistant way
    volumes:
      - ${VOLUME_PATH}:/var/lib/postgresql/data

    # Create a network called internal to allow the different containers present on that network to communicate
    networks:
      - internal

    restart: unless-stopped # Will restart unless manualy stopped

  backend:
    build:
      context: ./Backend/simpleapi
    image: backend
    depends_on:
      - database #Start database first
    env_file:
      - .env
      
      # Connected to two different networks so that it can make the bridge between the database and the frontend without connecting them directly.
    networks:
      - internal
      - public
    restart: on-failure:3 #Will try to restart 3 times if it fails

  httpd:
    build:
      context: ./Http-server
    image: frontend

    # Specific container name because diffent from "httpd"
    container_name: frontend

    # Specify the port where we can acess the app
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - public
    restart: no 

networks:
  internal:
    name: internal-network
    driver: bridge
  public:
    name: public-network
    driver: bridge
```

### 1-9 Document your publication commands and published images in dockerhub.

First I login with
```
docker login
```
Then I taged my image
```
tag database dorianbqn/tp1-database:1.0
tag backend dorianbqn/tp1-backend:1.0
tag frontend dorianbqn/tp1-frontend:1.0
```
Then I pushed
```
docker push dorianbqn/tp1-database:1.0
docker push dorianbqn/tp1-backend:1.0
docker push dorianbqn/tp1-frontend:1.0
```
### 1-10 Why do we put our images into an online repo?

The online repo is used to store share and import images easily.

## TP2 Github Actions

### 2-1 What are testcontainers?

Testcontainers is a Java library that helps to run Docker containers for testing. It makes it easy to automatically start services like databases for example.

### 2-2 For what purpose do we need to use secured variables ?

We use secured variables to keep sensitive information private. Logins for Docker Hub for example should never be exposed in the code or logs, especially in public repositories.

### 2-3 Why did we put needs: build-and-test-backend on this job? Maybe try without this and you will see!

"needs: test-backend" creates a dependency between jobs. It means that the "build-and-push-docker-image" job will only run after the tests pass in the "test-backend" job. If we don’t put this, the build job might run even if tests fail, leading to push broken code.

### 2-4 For what purpose do we need to push docker images?

Here we only push the images if the commit is from the main branch so it is supposed to be a stable version. This way we can always retreive the latest stable version from Docker hub if needed.

## TP3 Ansible

### 3-1 Document your inventory and base commands

My folder inventories only contains setup.yml for the moment.

Here are the commands I used until this point:

I used mkdir to crate the different folders my-project, ansible and inventories:
```
sudo mkdir my-project
```
Then I created the setup.yml file with nano:
```
cd my-project/ansible/inventories
sudo nano setup.yml
```
Then I completed the setup file with the code provided:
```
all:
 vars:
   ansible_user: admin
   ansible_ssh_private_key_file: /home/dbouquin/id_rsa
 children:
   prod:
     hosts: dorian.bouquin.takima.cloud

```
And finally I ran the different commands provided:
```
cd ..
ansible all -i inventories/setup.yml -m ping
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become
```

### 3-2 Document your playbook

Here is my playbook file:
```
- hosts: all
  gather_facts: true
  become: true

  roles:
    - docker
```
Basically, we just created a role called docker and in its folder task/main.yml we put the tasks that were previously in playbook.yml.
Here is the main.yml from roles/docker/tasks:
```
---
# tasks file for roles/docker
# Install prerequisites for Docker
    - name: Install required packages
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
          - python3-venv
        state: latest
        update_cache: yes

    # Add Docker’s official GPG key
    - name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/debian/gpg
        state: present

    # Set up the Docker stable repository
    - name: Add Docker APT repository
      apt_repository:
        repo: "deb [arch=amd64] https://download.docker.com/linux/debian {{ ansible_facts['distribution_release'] }} stable"
        state: present
        update_cache: yes

    # Install Docker
    - name: Install Docker
      apt:
        name: docker-ce
        state: present

    # Install Python3 and pip3
    - name: Install Python3 and pip3
      apt:
        name:
          - python3
          - python3-pip
        state: present

    # Create a virtual environment for Python packages
    - name: Create a virtual environment for Docker SDK
      command: python3 -m venv /opt/docker_venv
      args:
        creates: /opt/docker_venv  # Only runs if this directory doesn’t exist

    # Install Docker SDK for Python in the virtual environment
    - name: Install Docker SDK for Python in virtual environment
      command: /opt/docker_venv/bin/pip install docker

    # Ensure Docker is running
    - name: Make sure Docker is running
      service:
        name: docker
        state: started
      tags: docker
```
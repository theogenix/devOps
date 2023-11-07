# DevOps README

## Docker

### 1-1 Documement your database container essentials commands and Dockerfile

on utilise l'image postgres version 14.1  
`FROM postgres:14.1-alpine`

on initialise les variables de la db  
`ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd`

on copie nos deux fichiers SQL de création de tables et de remplissage de la table  
`COPY CreateScheme.sql /docker-entrypoint-initdb.d/  
COPY InsertData.sql /docker-entrypoint-initdb.d/`


### 1-2 Why do we need a multistage build? And explain each step of this dockerfile.

A multistage build in Docker involves using multiple FROM statements in a single Dockerfile. Each FROM statement starts a new stage and can have its own base image and set of instructions. This allows you to separate the build environment from the runtime environment, resulting in a smaller and more efficient final image.

*Step 1 (myapp-build):*

Uses the Maven image with Amazon Corretto 17 as the base image and names this step 'myapp-build'.  
Sets an environment variable MYAPP_HOME to /opt/myapp.  
Sets the working directory to /opt/myapp.  
Copies the pom.xml file from the current directory to the working directory.  
Copies the entire src directory from the current directory to /opt/myapp/src.  
Executes Maven to package the application, skipping the tests.  

*Step 2:*

Starts a new step with Amazon Corretto 17 as the base image.  
Redefines the environment variable MYAPP_HOME to /opt/myapp (redundant, but ensures consistency).  
Sets the working directory to /opt/myapp.  
Copies the JAR file(s) from the previous step (myapp-build) to /opt/myapp  

### 1-3 Document docker-compose most important commands.
`version`: Specifies the Docker Compose version being used.
`services`: Lists the deployed services in the form of containers.
`build`: Indicates the directory to be built.
`container_name`: Sets the name of the container.
`networks`: Specifies which network the service is connected to. Facilitates communication between containers.
`depends_on`: Specifies the dependency between services. If the specified service is not running, the current service will not start.
`ports`: Defines the ports that will be exposed by the container.

### 1-4 Document your docker-compose file.

`version: '3.7'`: Sets the Docker Compose specification version.

`services:` Defines the services that will be deployed as Docker containers

`build`: Indicates that the Docker image for the service will be built locally from a Dockerfile. This allows customization of the service's environment.  

`networks`: This command allows specifying which network the service is connected to. It is important for enabling communication between containers.  

`depends_on:` This command specifies dependencies between services. This means the specified service must be running before the current service is started.  

`ports`: Allows specifying which ports will be exposed by the container.

`my-network`: Defines a custom network named "my-network". Services can be connected to this network to facilitate communication between them.

`backend:, database:, httpd`: These are service names. Each service represents a component of the application.

The sections under each service (such as build, networks, etc.) need to be filled in with specific information to properly configure each service.

### 1-5 Document your publication commands and published images in Docker Hub.

`docker tag my-database USERNAME/my-database:1.0`

This command creates a new label for the Docker image my-database. The created label is USERNAME/my-database:1.0, where USERNAME is your Docker Hub account username, and 1.0 is the version of the image. This label helps differentiate different versions of the image.

`docker push USERNAME/my-database:1.0  `

Once the image has been labeled with a meaningful version (in this case, 1.0), you can push the image to your associated Docker Hub registry by using the commande above.

### gotheo78/tp1-httpd:1.0

- **Description:** This is a Docker image named `tp1-httpd` owned by the user `gotheo78` on Docker Hub. The version of this image is `1.0`.
- **Status:** Inactive (unused for some time).
- **Architecture:** Linux.
- **Size:** 64.76MB

### gotheo78/tp1-backend:1.0

- **Description:** This is a Docker image named `tp1-backend` owned by the user `gotheo78` on Docker Hub. The version of this image is `1.0`.
- **Status:** Inactive (unused for some time).
- **Architecture:** Linux.
- **Size:** 249.99MB.

### gotheo78/tp1-database

- **Description:** This is a Docker image named `tp1-database` owned by the user `gotheo78` on Docker Hub.
- **Status:** Inactive (unused for some time).
- **Architecture:** Linux.
- **Size:** 81.73MB


## GitHub Actions

### 2-1 What are testcontainers?

Testcontainers is a valuable tool for developers looking to automate tests for applications that rely on external services, making it easier to achieve reliable and comprehensive test coverage.

### 2-2 Document your GitHub Actions configurations.
```
name: CI devops 2023  # Name of this CI/CD workflow.

on:  
  push:  
    branches:  
      - main  # This workflow will be triggered on any push to the "main" branch.

jobs:

  test-backend:  # Definition of a job named "test-backend".
    runs-on: ubuntu-22.04  # This job will run on an Ubuntu 22.04 virtual machine.

    steps:  # Steps to be executed in this job.
      - uses: actions/checkout@v2.4.2  # Fetches source code from the repository.
      - name: Set up JDK 17
        uses: actions/setup-java@v3  # Sets up the Java environment.
        with:
          distribution: "adopt"      
          java-version: "17"         

      - name: Build and test with Maven
        run: mvn clean verify
        working-directory: backend/simpleapi/simple-api-student  # Specifies the working directory for this step.

  build-and-push-docker-image:
    needs: test-backend  # This job depends on "test-backend" being executed first.
    runs-on: ubuntu-22.04  # This job will run on an Ubuntu 22.04 virtual machine.

    steps:  # Steps to be executed in this job.
      - name: Login to DockerHub
        run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}  # Logs in to DockerHub.

      - name: Checkout code
        uses: actions/checkout@v2.5.0  # Fetches source code from the repository.

      - name: Build and push backend image
        uses: docker/build-push-action@v3  # Builds and pushes the backend Docker image.
        with:
          context: backend/simpleapi/simple-api-student
          push: ${{ github.ref == 'refs/heads/main' }}  # Pushes only if the branch is "main".
          tags: ${{secrets.DOCKER_USERNAME}}/tp-devops-simple-api-backend:latest  # Image tag.

      - name: Build and push database image
        uses: docker/build-push-action@v3  # Builds and pushes the database Docker image.
        with:
          context: database
          push: ${{ github.ref == 'refs/heads/main' }}  # Pushes only if the branch is "main".
          tags: ${{secrets.DOCKER_USERNAME}}/tp-devops-simple-api-database:latest  # Image tag.

      - name: Build and push httpd image
        uses: docker/build-push-action@v3  # Builds and pushes the httpd Docker image.
        with:
          context: http
          push: ${{ github.ref == 'refs/heads/main' }}  # Pushes only if the branch is "main".
          tags: ${{secrets.DOCKER_USERNAME}}/tp-devops-simple-api-httpd:latest  # Image tag.
```


### 2-3 Document your quality gate configuration.

```
name: CI devops 2023  # Name of this CI/CD workflow.

on:
  push:
    branches:
      - main  # This workflow will be triggered on any push to the "main" branch.

jobs:

  test-backend:  # Definition of a job named "test-backend".
    runs-on: ubuntu-22.04  # This job will run on an Ubuntu 22.04 virtual machine.

    steps:  # Steps to be executed in this job.
      - uses: actions/checkout@v2.4.2  # Fetches source code from the repository.
      - name: Set up JDK 17
        uses: actions/setup-java@v3  # Sets up the Java environment.
        with:
          distribution: "adopt"      
          java-version: "17"         

      - name: Build and test with Maven
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=theogenix_devOps -Dsonar.organization=sonarcloudtheogenix -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file ./pom.xml
        working-directory: backend/simpleapi/simple-api-student  # Specifies the working directory for this step.

  build-and-push-docker-image:
    needs: test-backend  # This job needs "test-backend" to be executed before it.
    runs-on: ubuntu-22.04  # This job will run on an Ubuntu 22.04 virtual machine.

    steps:  # Steps to be executed in this job.
      - name: Login to DockerHub
        run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}  # Logs in to DockerHub.

      - name: Checkout code
        uses: actions/checkout@v2.5.0  # Fetches source code from the repository.

      - name: Build and push backend image
        uses: docker/build-push-action@v3  # Builds and pushes the backend Docker image.
        with:
          context: backend/simpleapi/simple-api-student
          push: ${{ github.ref == 'refs/heads/main' }}  # Pushes only if the branch is "main".
          tags: ${{secrets.DOCKER_USERNAME}}/tp-devops-simple-api-backend:latest  # Image tag.

      - name: Build and push database image
        uses: docker/build-push-action@v3  # Builds and pushes the database Docker image.
        with:
          context: database
          push: ${{ github.ref == 'refs/heads/main' }}  # Pushes only if the branch is "main".
          tags: ${{secrets.DOCKER_USERNAME}}/tp-devops-simple-api-database:latest  # Image tag.

      - name: Build and push httpd image
        uses: docker/build-push-action@v3  # Builds and pushes the httpd Docker image.
        with:
          context: http
          push: ${{ github.ref == 'refs/heads/main' }}  # Pushes only if the branch is "main".
          tags: ${{secrets.DOCKER_USERNAME}}/tp-devops-simple-api-httpd:latest  # Image tag.

```
## Ansible

### 3-1 Document your inventory and base commands
`all`: This is the root level of the YAML document. It indicates the beginning of the inventory file.

`vars`: This section is used to define variables that can be used in Ansible playbooks. In this case, two variables are defined:

`ansible_user`: Specifies the username (centos) that Ansible will use to connect to the target host.

`ansible_ssh_private_key_file`: Specifies the file path (/etc/ansible/id_rsa) to the private key that Ansible will use for authentication.

`children`: This section is used to group hosts into different categories, referred to as "children". These categories can then be used to apply configurations or tasks to specific groups of hosts.

`prod`: This is the name of the group of hosts. In this case, it is named prod.

`hosts`: Indicates that the following line will list the hosts belonging to the prod group.

`theo-masaki.genix.takima.cloud`: This is the hostname of the target server. It belongs to the prod group.

### 3-2 Document your playbook

`hosts`: all: This playbook will apply to all hosts listed in the inventory file. It targets all available hosts.

`gather_facts`: false: This disables the gathering of facts about the target hosts. Facts include information like IP addresses, operating system details, and more. Disabling this can make the playbook run faster if you don't need these facts.

`become`: true: This indicates that Ansible should use privilege escalation (such as sudo) to execute tasks. This allows tasks to be executed with elevated privileges.

`roles`: This section specifies the roles that will be applied to the hosts. Roles are a way to organize and reuse sets of tasks, handlers, and variables.

`docker`: This role will be applied to the hosts. It likely includes tasks related to installing and configuring Docker.

`network`: This role will be applied to the hosts. It likely includes tasks related to network configuration.

`backend`: This role will be applied to the hosts. It likely includes tasks related to setting up the backend of an application.

`database`: This role will be applied to the hosts. It likely includes tasks related to setting up and configuring a database.

`httpd`: This role will be applied to the hosts. It likely includes tasks related to setting up and configuring an HTTP server.

### 3-3 Document your docker_container tasks configuration.

```
- name: launch backend   # This is a task named "launch backend" which describes the action to be performed.
  docker_container:    # This module is used to manage Docker containers.
    name: simpleapistudent3   # The name of the Docker container to be managed.
    image: gotheo78/tp-devops-simple-api-backend:latest   # The Docker image to use for this container.
    state: started   # The desired state of the container, in this case, it's set to "started".
    networks:    # This section is used to configure the networks the container will be connected to.
      - name: app-network   # The name of the network the container will be connected to.

```
```
- name: launch database   # This is a task named "launch database" which describes the action to be performed.
  docker_container:    # This module is used to manage Docker containers.
    name: tp1_docker   # The name of the Docker container to be managed.
    image: gotheo78/tp-devops-simple-api-database:latest   # The Docker image to use for this container.
    state: started   # The desired state of the container, in this case, it's set to "started".
    networks:    # This section is used to configure the networks the container will be connected to.
      - name: app-network   # The name of the network the container will be connected to.
```
```

# Install device-mapper-persistent-data
- name: Install device-mapper-persistent-data
  yum:
    name: device-mapper-persistent-data
    state: latest

# Install lvm2
- name: Install lvm2
  yum:
    name: lvm2
    state: latest

# Add Docker repository
- name: add repo docker
  command:
    cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

# Install Python3
- name: Install Python3
  yum:
    name: python3
    state: latest

# Install Python pip
- name: Install Python pip
  yum:
    name: python3-pip
    state: latest

# Install Docker
- name: Install Docker
  yum:
    name: docker-ce
    state: present

# Make sure Docker is running
- name: Make sure Docker is running
  service: name=docker state=started
  tags: docker
```
```
- name: launch proxy
  docker_container:
    name: proxy   # Name of the Docker container.
    image: gotheo78/tp-devops-simple-api-httpd:latest   # Docker image to use for the container.
    state: started   # Desired state of the container (started).
    ports:
      - "80:80"   # Port mapping from host (left side) to container (right side).
    networks:
      - name: app-network   # Network configuration for the container.
```
```

- name: create network
  docker_network:
    name: app-network   # Name of the Docker network to be created.
    driver: bridge      # Specifies the network driver to be used (bridge in this case).

```



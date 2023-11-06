# Docker and GitHub Actions README

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

*Build Stage:*
1. A Maven-based image is used to build the application.
2. The pom.xml and source code are copied into the container.
3. Maven is used to compile and package the application, producing a JAR file.

*Run Stage:*
4. An Amazon Corretto 17 image is used for the runtime.
5. The JAR file built in the previous stage is copied to the runtime image.
6. The entry point is set to run the Java application from the JAR file.

This separation reduces the final image size and enhances security by excluding unnecessary build tools and artifacts in the runtime image.

### Why should we run the container with a flag -e to give the environment variables?

Using the -e option to set environment variables when running a Docker container is essential for configuring the application in a flexible, secure, and portable way. This separates configuration from application data, promotes consistency between environments, and follows DevOps best practices.

### Why do we need a volume to be attached to our postgres container?

We need a volume linked to the PostgreSQL container to ensure that data is preserved even when the container is destroyed.

### 1-3 Document docker-compose most important commands.

- docker-compose up:
  - *Description:* Start containers defined in the docker-compose.yml file.
  - *Usage:* docker-compose up

- docker-compose restart:
  - *Description:* Restart containers defined in the docker-compose.yml file.
  - *Usage:* docker-compose restart

- docker-compose down:
  - *Description:* Stop and remove containers defined in the docker-compose.yml file.
  - *Usage:* docker-compose down

- docker-compose pull:
  - *Description:* Pull service images defined in the docker-compose.yml file without starting containers.
  - *Usage:* docker-compose pull

- docker-compose ps:
  - *Description:* List containers associated with the project defined in the docker-compose.yml file.
  - *Usage:* docker-compose ps

- docker-compose up -d:
  - *Description:* Start containers in detached mode, running them in the background.
  - *Usage:* docker-compose up -d

- docker-compose logs:
  - *Description:* View the logs of the containers.
  - *Usage:* docker-compose logs

- docker-compose -f [file]:
  - *Description:* Specify an alternative Compose file (other than the default docker-compose.yml) to use.
  - *Usage:* docker-compose -f [file] [command]

- docker-compose exec:
  - *Description:* Run a command in a running container.
  - *Usage:* docker-compose exec [service] [command]

- docker-compose build:
  - *Description:* Build or rebuild services defined in the docker-compose.yml file.
  - *Usage:* docker-compose build

### 1-4 Document your docker-compose file.

*Version:* The Docker Compose file is defined using version 3.7.

*Services:* This section defines the individual components of the application as services.

- *backend:*
  - This service is responsible for the backend application.
  - It is built from the source code located in the ./backend/simpleapi/simple-api-student directory.
  - The container_name specifies the name of the container as "backend."
  - It is part of the app-network to facilitate communication with other services.
  - It depends on the database service, ensuring that the database is up and running before the backend starts.

- *database:*
  - This service represents the database component of the application.
  - The container_name sets the name to "database."
  - The service is built from the ./database directory.
  - It also belongs to the app-network for inter-service communication.

- *httpd:*
  - This service represents an HTTP server (e.g., Apache HTTP server).
  - The container is built from the ./http directory.
  - Port mapping is defined, with host port 8081 mapped to container port 80.
  - It's part of the app-network for network communication.
  - The depends_on section ensures that the httpd service starts only after the backend service is up and running.

*Networks:*
- The app-network is a custom Docker network created to allow the services to communicate with each other. It provides isolation and connectivity for the services within the application.

### 1-5 Document your publication commands and published images in Docker Hub.

- docker tag my-database USERNAME/my-database:1.0:
  - Tags an existing Docker image as USERNAME/my-database:1.0.

- docker push USERNAME/my-database:1.0:
  - Pushes the tagged image to Docker Hub for publication.

These commands are used to tag and publish Docker images to your Docker Hub repository.

## GitHub Actions

### What is it supposed to do? → mvn clean verify

The command mvn clean verify is used to build and run tests in a Maven-based Java project. It cleans the project, compiles the code, and executes tests to ensure the application functions correctly. It should be run from the project's root directory where the pom.xml file is located.

### 2-1 What are testcontainers?

Testcontainers is a Java library used for simplified integration testing by managing lightweight, disposable containers. It's particularly useful for starting and stopping services like databases during testing, ensuring test reliability.

### 2-2 Document your GitHub Actions configurations.

*GitHub Actions configuration summary:*

- *Workflow name:* "CI devops 2023"
- *Triggered by:* Pushes to the master branch and pull requests.
- *Job:* "test-backend" on ubuntu-22.04.
- *Steps:* Check out code, set up JDK 17, build and test with mvn clean verify.
- *Secrets:* DOCKERHUB_USERNAME and DOCKERHUB_TOKEN for secure access.
- **Integration testing and possible CI/CD pipeline.

These configurations automate building, testing, and potentially deploying your application while safeguarding secrets for security.

### For what purpose do we need to push Docker images?

Docker images are pushed for distribution, deployment, version control, and portability. They facilitate collaboration, support CI/CD, and ensure that the latest application version is available for use.

### Document your quality gate configuration.

Quality gate configuration using SonarCloud involves:

- Register on SonarCloud and create an organization.
- Retrieve project and organization keys.
- Modify your GitHub Actions workflow to run SonarCloud analysis after building and testing.
- Set parameters like sonar.projectKey and sonar.organization.
- Use the secrets.SONAR_TOKEN for authentication.
- Ensure code quality and security with each commit and view SonarCloud analysis reports online.

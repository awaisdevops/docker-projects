# Docker Application Setup with AWS ECR & Nexus

This project sets up a Node.js application using Docker, building a Docker image, running containers with `docker-compose`, and pushing and pulling images to AWS ECR and Nexus repositories.

## Clone the Application Code Repository

Clone the application repository from GitHub:

```bash
git clone https://github.com/awaisdevops/docker-projects.git
cd docker-composr-images-aws-ecr-nexus
```

## Package the Application into a Docker Image

### Create the `Dockerfile`

Create a `Dockerfile` with the following content:

```dockerfile
FROM node:13-alpine

ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PWD=password

RUN mkdir -p /home/app

COPY ./app /home/app

# Set default dir so that next commands execute in /home/app dir
WORKDIR /home/app

RUN npm install

# No need for /home/app/server.js because of WORKDIR
CMD ["node", "server.js"]
```

### Build the Docker Image

Use the `Dockerfile` to build the Docker image:

```bash
docker build -t my-app:1.0 .
```

## Create `docker-compose.yaml`

Create a `docker-compose.yaml` file with the following content:

```yaml
version: '3'
services:
  my-app:
    image: my-app:1.0
    ports:
      - 3000:3000
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db
  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
    depends_on:
      - "mongodb"
volumes:
  mongo-data:
    driver: local
```

### Start All Containers with `docker-compose`

Run the following command to start all containers:

```bash
docker-compose -f docker-compose.yaml up
```

Access the following URLs to interact with the services:

- MongoDB UI: [http://localhost:8081](http://localhost:8081)
- Application: [http://localhost:3000](http://localhost:3000)

- ![Image 1](assets/image1.png)
- ![Image 2](assets/image2.png)

## Push and Pull Docker Image to AWS ECR

### Create an AWS ECR Repository

1. Go to the AWS Management Console > Services > ECR.
2. Create a new repository by configuring the settings.

You can also use the following command to create an ECR repository from the AWS CLI:

```bash
aws ecr create-repository --repository-name <repository_name> --region <region>
```

For example:

```bash
aws ecr create-repository --repository-name my-repository --region us-west-2
```

### Authenticate Docker to AWS ECR

Before pushing the image, ensure that you have configured the AWS CLI.

Run the following command to authenticate Docker to your AWS ECR:

```bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

### Tag the Docker Image

Tag your Docker image for AWS ECR:

```bash
docker tag my-app:latest 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-repository:latest
```

### Push the Docker Image to AWS ECR

Push the image to your AWS ECR repository:

```bash
docker push 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-repository:latest
```

### Modify compose file for AWS ECR

Update your compose file to use the AWS ECR image:

```yaml
version: '3'
services:
  my-app:
    image: 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-repository:latest
    ports:
      - 3000:3000
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db
  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
    depends_on:
      - "mongodb"
volumes:
  mongo-data:
    driver: local
```

Now, run the containers using `docker-compose`:

```bash
docker-compose -f aws-ecr-docker-compose.yaml up
```

## Push and Pull Docker Image to Nexus

### Create a Docker Repository in Nexus

1. Create a Docker repository in Nexus.
2. Configure the repository's port and access settings.
3. Create roles with permissions and users to access the repository.

### Configure Docker to Trust Nexus Registry

Edit `/etc/docker/daemon.json` to allow Docker to trust the Nexus registry:

```json
{
  "insecure-registries": ["<nexus_host>:<docker_port>"]
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

### Login to Nexus

Log in to Nexus using Docker:

```bash
docker login <nexus_host>:<docker_port>
```

### Tag the Docker Image

Tag your Docker image for Nexus:

```bash
docker tag my-app:1.0 mycompany.nexus.com:8083/my-docker-repo/my-app:1.0
```

### Push the Docker Image to Nexus

Push the image to your Nexus repository:

```bash
docker push mycompany.nexus.com:8083/my-docker-repo/my-app:1.0
```

### Pull the Docker Image from Nexus

You can pull the image from your Nexus repository using:

```bash
docker pull mycompany.nexus.com:8083/my-docker-repo/my-app:1.0
```

### Modify compose for Nexus

Update your compose to use the Nexus image:

```yaml
version: '3'
services:
  my-app:
    image: mycompany.nexus.com:8083/my-docker-repo/my-app:1.0
    ports:
      - 3000:3000
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db
  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
    depends_on:
      - "mongodb"
volumes:
  mongo-data:
    driver: local
```

Now, run the containers using `docker-compose`:

```bash
docker-compose -f nexus-docker-compose.yaml up
```

# Refactor Udagram

Project 3 required refactoring of the **Udagram Project** from [Udacity Cloud Developer Nanodegree](https://www.udacity.com/course/cloud-developer-nanodegree--nd9990) from a *monolithic* app to a *mircoservice architecture*

# Project Dependencies

To deploy the project locally you will need the following:

* [Node.js](https://docs.npmjs.com/try-the-latest-stable-version-of-node) and [NPM](https://docs.npmjs.com/try-the-latest-stable-version-of-npm)
* [Ionic](https://ionicframework.com/docs/intro/cli)
* [Docker-compose](https://docs.docker.com/compose/install/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* [AWS Account](https://aws.amazon.com) (PostgreSQL DB and a S3 Bucket)

# Setup Local Environment
## Run Server-Side Services
To run server-side services, open a service from your favorite terminal and execute:
```
npm install
```
Then, run the application with:
```
npm run dev
```

For a successful run, you should edit your `~/.profile` to add this values:
```
export POSTGRESS_USERNAME="your_db_username";
export POSTGRESS_PASSWORD="your_db_password";
export POSTGRESS_DB="your_db_name";
export POSTGRESS_HOST="your_db_host";
export AWS_REGION="your_aws_region";
export AWS_PROFILE="your_aws_profile";
export AWS_BUCKET="your_s3_bucket";
export JWT_SECRET="your_jwt_secret"
```

## Run Frontend Application
To run our Ionic application, first we are going to install dependencies:
```
npm install
```
Then, to run the development server, open terminal and run:
```
ionic serve
```

# Setup Containers With Docker-Compose

In each project, you can find the Dockerfile, to create image you can run:
```
docker build .
```
For tagging images you can use:
```
docker tag source-image:source-version target-image:target-version
```
Upload images to your own docker hub account with `docker push`, don't forget to tag with your user name. 

When you have all image up, next step is edit image names in docker-compose-build.yaml.
Finally, turn on your environment with:
```
docker-compose up
```

# Setup a Local Cluster
Once you have your public images and a Minikube instance or similar, you are ready to deploy your pods in Kubernetes.
First, you can edit .yaml files (they are in /udacity-c3-deployment/k8s) to place them your own images and secrets and properties.
Deploy configmaps and secrets:
```
kubectl apply -f aws-secret.yaml
kubectl apply -f env-configmap.yaml
kubectl apply -f env-secret.yaml
```
Then, apply deployments:
```
kubectl apply -f backend-feed-deployment.yaml
kubectl apply -f backend-user-deployment.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f reverseproxy-deployment.yaml
```
You can check the health of your pods with:
```
kubectl get pods
```

Then, apply service yamls:
```
kubectl apply -f backend-feed-service.yaml
kubectl apply -f backend-user-service.yaml
kubectl apply -f frontend-service.yaml
kubectl apply -f reverseproxy-service.yaml
```
Finally, using `kubectl port-forwarding` with your reverseproxy and frontend services, you can see your app running locally from your cluster.

# TravisCI Integration
This repository has a TravisCI integration. Every merge caused the start of a deployment.
Travis configuration file (.travis.yml) looks like this:
```yaml
language: minimal

services: docker

env:

- DOCKER_COMPOSE_VERSION=1.23.2

before_install:

- docker -v && docker-compose -v

- sudo rm /usr/local/bin/docker-compose

- curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > docker-compose

- chmod +x docker-compose

- sudo mv docker-compose /usr/local/bin

- curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

- chmod +x ./kubectl

- sudo mv ./kubectl /usr/local/bin/kubectl

install:

- docker-compose -f udacity-c3-deployment/docker/docker-compose-build.yaml build --parallel
``` 
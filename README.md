# DevOps_Project_FMI
Final project for the DevOps course @ Faculty of Mathematics and Informatics, Sofia University

## Branching Strategy
The project has a simple branching strategy:
* main - stable branch
* feature/* - feature branches

## Docker
The project uses Docker to containerize the aplication.
### Dockerfile
Base image: python:3.12-slim
Creates working dir /app
Copies requirements.txt and installs dependencies
Copies the source code(src) into /app
Exposes port 5000
Container start command: python app.py
### Docker image
The docker image is build with this command: 
docker build -t ilianamiladinova/devops_project_flask:0.0.2 .
And is pushed to Docker Hub with this:
docker push ilianamiladinova/devops_project_flask:0.0.2
Start the container in Cloud Shell:
docker run -d -p 5000:5000 --name devops-flask ilianamiladinova/devops_project_flask:0.0.2  

## Kubernetes
The project deploys the application to kubernetes by using manifest or a Helm chart
### The kubernetes manifest are located in k8s/ :
* deployment.yaml
    * Defines a deployment
    * name: devops-project-flask-deployment
    * two replicas
    * container image: ilianamiladinova/devops_project_flask:0.0.2
    * container port: 5000
    * readinessProbe and livenessProbe use /status endpoint
* service.yaml
    * Defines NodePort service 
    * name: devops-project-flask-service
    * selector app: devops-project-flask
    * Service port: 80
    * Target port: 5000

To apply the kubernetes manifests run the following commands:
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
Verify that the pods are running by running this command:
kubectl get pods
Check the service: kubectl get svc

To access the application in Cloud Shell we use post-forwarding:
kubectl port-forward service/devops-project-flask-service 8080:80

### Helm Chart
* Located in helm/devops-project
* Uses values.yaml to configure:
    * Image repository and tag
    * Service type and port
    * replicas
    * liveness and readiness probe

* Validate syntax:
helm lint helm/devops-project
* Deploy:
helm install devops-app ./devops-project        
or if we already have an installation:
helm upgrade devops-app ./devops-project

To access the application in Cloud Shell we use post-forwarding:
kubectl port-forward service/devops-app-devops-project-service 8080:80

### CI/CD pipline 
The pipeline is located in .github/workflows/pipeline.yml. It is triggered on push and pull requests events on the main branch and feature/* branches.
#### CI 
* ensures code quality and security before building the artifact
##### Jobs:
* Flake8 - runs flake8 to check Python syntax
* Build and Test:
    * installs python dependences
    * runs unittest(with pytest)
* SAST scan
    * performs a SAST
    * scans the source code for security issues with Bandit
* Docker 
    * Builds the Docker image.
    * Pushes the image to Docker Hub
#### CD
After the code passes security checks we use Infrastructure as Code to deploy the application
##### Jobs:
* Helm validation - validates the helm chart and renders the kubernetes manifest with helm template
* Deploy:
    * Creates a temporary Kubernetes cluster with kind
    * Deploys the application with Helm
    * Verifies the deployment 

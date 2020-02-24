# Author
Rajesh Sapkale

# Introduction
This is Jenkins CICD Pipeline building SpringBoot application on slave container and creating docker image, also pushing it to container registry.

# Jenkinsfile
Git checkout -> Maven Build -> Unit Testing -> SonarQube Integration -> Artifactory Integration -> Docker Image creation and pushing to google container registry

# K8sDeployment
K8s Deployment using docker container from google container registry

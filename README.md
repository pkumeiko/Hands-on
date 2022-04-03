![Build Status](http://54.171.122.152:8080/buildStatus/icon?job=My-pipeline)

## Hands-on assignment
### The repo contains the following elements
1. At root level spring-boot ``perso-greet`` project ( [tutorial](https://spring.io/guides/gs/serving-web-content/) ) 
   * Maven project - build using `mvn clean package`
   * Dockerfile - Packaging this project in Image  
   * Jenkinsfile - CI/CD for this project  
2. helm-package
   * Contains greet chart - basic web helm package to wrap the docker image container for k8s deployment
   * Dockerfile_helm - docker image for building the helm package  
3. deployment folder 
   * containing ansible(v2.9) playbooks and roles 
     - Setup the Jenkins Master server
     - Setup the k8s (minikube) deployment server
     - Deploying the app via helm to k8s instance (localhost minikube)
     - Starting + Stopping port-forward from remote to the minikube service (work-in-progress)
4. environment folder 
   * Containing the info to access both servers 
    
## Usage
1. File "deployment/inventory" should be changed with your EC-2 IP addresses
2. To deploy Jenkins to EC-2 instance: ansible-playbook deployment/deploy_jenkins.yml -i deployment/inventory
3. To deploy Minikube to EC-2 instance: ansible-playbook deployment/deploy_minikube.yml -i deployment/inventory

## Setting up Jenkins
#### 1. Login on http://jenkins-instance-ip-address:8080 with default credentials
#### 2. Create Pipeline: New Item -> Pipeline 
#### 3. Following settings shall be checked during pipeline creation:
   1. Github project -> Project URL (example https://github.com/pkumeiko/Hands-on/)
   2. Build Triggers -> GitHub hook trigger for GITScm polling
   3. Pipeline -> Pipeline script from SCM -> SCM (Git) -> Repository URL (example https://github.com/pkumeiko/Hands-on.git) -> Credentials (add your username\password or SSH key for Github auth to be download repo)
   4. Branches to build -> Branch specifier (*/main)
   5. Script patch (Jenkinsfile)
#### 4. Setup Maven in Jenkins:   Manage Jenkins -> Global Configuration Tool -> Add Maven -> Maven:3.8.5 from Apache
#### 5. Create ec2 credentials for accessing Minikube instance from Jenkins: Manage Jenkins -> Manage Credentials -> click on domain "global" -> Add credentials:
   1. Kind "SSH username with private key"
   2. ID: ec2
   3. Username: ec2-user
   4. Private key -> Enter directly -> paste your private SSH key for minikube EC2-instance
#### 6. Setup webhook on your cloned repo: Settings -> Webhooks -> add webhook -> Payload URL (http://<jenkins-instance-ip-address>:8080/github-webhook) -> Content type (json)
#### 7. Following groovy script should be approved in Jenkins (to run and build greet-app): Manage Jenkins -> In-process Script Approval ->
  ```    
  staticMethod org.codehaus.groovy.runtime.DefaultGroovyMethods getAt java.lang.Object java.lang.String
    staticMethod org.codehaus.groovy.runtime.DefaultGroovyMethods putAt java.lang.Object java.lang.String java.lang.Object
```
  
#### 8. You can also setup email provider to send notifications with build status:
  * Manage Jenkins -> Configure System -> 
  1. System Admin e-mail address (type-in email address which will be used to send notifications <from:>)
  2. Extended E-mail Notification (preinstalled) -> <your SMTP server address> <port 465> <add email credentials username\password> <use SSL>
  3. E-mail Notification (has to be filled-in to work properly with SMTP) -> <Use SMTP Auth><username and password><use SSL><SMTP port 465>
  
# After all these steps Jenkins should be ready to receive your recent changes on Github and deploy it to EC2-instance
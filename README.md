# Task: Automate Deployment of a Simple Web Application

## Prerequisites :-

1. **Launch the EC2 Instance :**

   Launch Ubuntu EC2 instance and allocate the Elastic IP to that instance so public IP of that instance doesn’t change while restart the instance.

1. **Install Jenkins :**
   
   Ensure Jenkins is installed on your EC2 instance by Using below official documentation.

   <https://www.jenkins.io/doc/book/installing/linux/#debianubuntu> 

1. **Install Docker and Docker Compose**
1. **Set Up Docker Hub Account**
   1. Create an account at Docker Hub.
   1. Generate Docker Hub credentials to use in Jenkins.
1. **Install Node.js and npm**
   
   Ensure Node.js and npm are installed. Verify installation using:

       node -v
    
       npm -v

1. **Set Up Jenkins Credentials**
   
   Add Docker Hub credentials to Jenkins:
   1. Go to Manage Jenkins > Manage Credentials.
   1. Add a new Username and Password credential with:
      1. **ID**: dockerhub
      1. **Username**: Docker Hub username
      1. **Password**: Docker Hub password
   1. Go to Manage Jenkins > Tools and add the docker installation
1. **Add Jenkins User to the Docker Group**

       sudo usermod -aG docker jenkins
    
       sudo systemctl restart jenkins
   <br><br>



## Steps to Set Up and Run the Pipeline :

**1. Fork the github repository and make necessary changes:** 

- Clone the repository to your local machine:

      git clone https://github.com/rohan7739/testnode.git
      
      cd testnode

- then I have added the Dockerfile, docker-compose.yaml file and push the changes.<br>


**2. Configure Jenkins Pipeline**

1. Log in to Jenkins.
1. Create a new pipeline job:
   1. Click New Item > Enter a name > Select Pipeline.
1. Configure the pipeline:
   1. Scroll down to the Pipeline section.
   1. Select Pipeline script.
   1. Paste the Jenkinsfile content (as provided below).

           pipeline {
              agent any
              stages {
                  stage('Git Code Checkout') {
                      steps {
                          git 'https://github.com/rohan7739/testnode.git'
                      }
                  }  
                  stage('Install Dependencies') {
                      steps {
                          sh 'npm install'
                      }
                  }      
                  stage('Build the Docker Image') {
                      steps {
                          sh "docker build -t node-app-test-new ."
                     }
                  }       
                  stage('Push to Docker Hub') {
                      steps {
                          withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPass', usernameVariable: 'dockerHubUser')]) {
                              sh "docker tag node-app-test-new ${env.dockerHubUser}/node-app-test-new:latest"
                              sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                              sh "docker push ${env.dockerHubUser}/node-app-test-new:latest"
                          }
                      }
                  }
                  stage("Deploy"){
                      steps{
                          sh "docker-compose up -d"
                      }
                  }
              }
          }

     ### NOTE :  After the first Build is done then change the last stage deploy with the following stage
            stage("Deploy"){
                        steps{
                            sh "docker-compose down && docker-compose up -d"
                        }
               }


**3.  Configure the Github webhook in pipeline**

![git_check](https://github.com/user-attachments/assets/4d3f8843-dd9e-492e-baa3-d63fff56b67b)

**4.  Install Plugins** <br>
  o	Go to Manage Jenkins > Plugins and install the plugins as below
  
  1.	Github Integration
  2.	Pipeline stage view
  3.	Generic webhook trigger plugins
  4.	Docker compose build step plugin
  5.	Docker pipeline


**5.  Trigger the Pipeline**

1. Save the pipeline configuration.
2. Trigger a build by clicking Build Now in Jenkins.<br>
      Or <br>
***NOTE :  Whenever changes are made to the code then pipeline automatically triggered and deploy the application by using Github-webhook***


-----
**Pipeline Stages Explained**

1. **Git Code Checkout**
   1. Clones the repository from GitHub to the Jenkins workspace.
1. **Install Dependencies**
   1. Installs the necessary npm dependencies using **npm install**.
1. **Build the Docker Image**
   1. Builds a Docker image for the Node.js application using the Dockerfile.
1. **Push to Docker Hub**
   1. Tags the Docker image and pushes it to your Docker Hub account using stored credentials.
1. **Deploy**
   1. Deploys the application using docker-compose:
      1. Stops any running containers (docker-compose down).
      1. Starts the containers in detached mode (docker-compose up -d).<br><br>












## Deployment Details

**Access the Application**

- Once deployed, the application should be accessible at:

      http://3.108.111.124:3000/

![main_pipeline_final](https://github.com/user-attachments/assets/563010a2-cf6c-4912-8c9d-baa8557b263e)


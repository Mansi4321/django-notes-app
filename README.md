Jenkins Declarative CI CD pipeline using AWS cloud
In this blog we will configure Jenkins declarative pipeline using AWS cloud

*   First of all, go to the AWS portal, and create a new instance. 

• Name: jenkins-server

•AMI: ubuntu.

•Instance type: t2.micro (free tier).

•Allow HTTP.

•Allow HTTPS.

Now, we will allow ports 8080 and 8000 for this machine from a security group. We can find the security group in the VM description. Now, here we need to allow “Inbound Rule”
* Now, connect to the EC2 instance that you have created.

Click on Launch Instance

•Update the server :

Once connected, update the server by running the following commands:

sudo apt update

git clone https://github.com/Mansi4321/django-notes-app.git

cd django-notes-app-with-database

sudo apt install git docker.io -y

sudo usermod -aG docker ubuntu

sudo reboot

Verify that Ubuntu has permission to run Docker by executing docker images command.

*Dockerfile:

Create a Dockerfile with the following content:

FROM python:3.9

WORKDIR /app/backend

COPY requirements.txt /app/backend

RUN pip install -r requirements.txt

COPY . /app/backend

EXPOSE 8000

CMD python /app/backend/manage.py

*Build the docker image by running following command :

docker build -t my-note-app .

docker images

Install Java JDK and Jenkins from following link:

https://www.jenkins.io/doc/book/installing/linux

*Grant Jenkins access to build and run Docker images:

sudo usermod -aG docker jenkins

sudo reboot

Access Jenkins by entering your EC2 instance's public IP followed by port 8080 in your browser. Obtain the initialAdminPassword by running:

cat /var/lib/jenkins/secrets/initialAdminPassword

Copy and paste the password in the provided tab and install all the necessary plugins.

* Creating a Declarative Pipeline

Now, let's create a declarative pipeline in Jenkins:

Click on "New Item" and name it "my-note-app-cicd-pipeline".

*Under the configuration settings, select the GitHub project and provide the repository URL:

https://github.com/Mansi4321/django-notes-app.git

Set the name to "note-app".

Enable the "Build Triggers" option by ticking the box next to "GitHub hook trigger for GITScm pooling".

In the "Pipeline" section, choose "Pipeline Script" and add the following script:

pipeline {

agent any

stages{

stage("Clone Code"){

steps {

echo "Cloning the code"
git url:"https://github.com/Mansi4321/django-notes-app.git", branch: "main"

}

}

stage("Build"){

steps {

echo "Building the code"

sh "docker build -t my-note-app ."

}

}

stage("Push to Docker Hub"){

steps {

echo "Pushing the image to Docker Hub"
withCredentials([usernamePassword(credentialsId:"dockerhub",passwordVariable:

dockerhubPass", usernameVariable:"dockerhubUser")]){

sh "docker tag my-note-app ${env.dockerhubUser}/my-note-app:latest"

sh "docker login -u ${env.dockerhubUser} -p ${env.dockerhubPass}"

sh "docker push ${env.dockerhubUser}/my-note-app:latest"

}

}

}
stage("Deploy"){

steps {

echo "Deploying the container"

sh "docker-compose down && docker-compose up -d "

}

}

}

}
*Before building, make sure to follow these additional steps:

1.Add DockerHub credentials in the global credentials section with the ID as "dockerHub" along with the username and password.

2.Create a Docker Compose file to enhance the deployment of Docker containers:

version : "3.3"

services :

web :

image : mansi4321/my-note-app:latest

ports : - "8000:8000"

Install Docker Compose before running the pipeline.

Once everything is set up, you can manually trigger the pipeline by clicking the build button in the Jenkins interface.

Your Note App is now deployed and ready to use. Enjoy the benefits of automation and continuous integration

To achieve full automation, configure webhooks so that any changes pushed to the GitHub repository will trigger the Jenkins pipeline and initiate the deployment process automatically.

* Further Simplification with "Pipeline Script from SCM"
* To make your pipeline more effective and easier to modify, you can use "Pipeline Script from SCM." This approach allows you to provide a GitHub repository link containing the Jenkinsfile. By adopting this method, any developer in your team with proper permissions can review and modify the Jenkinsfile, simplifying the pipeline management process

By leveraging webhooks and the "Pipeline Script from SCM" approach, your team can work collaboratively on the Jenkinsfile and make changes with ease, eliminating the need to access the Jenkins dashboard

That's it! You have now successfully configured a Jenkins declarative pipeline using AWS Cloud.




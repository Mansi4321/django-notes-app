pipeline{
    agent any 
    
    stages{
        stage("Clone Code"){
            steps {
                echo "Cloning the code"
                git url:"https://github.com/Mansi4321/django-notes-app.git", branch: "main"
            }
        }
        stage("build"){
            steps {
               echo "Building the code" 
               sh "docker build -t my-note-app ."
            }
        }
        stage("push to Docker Hub"){
            steps {
                echo "Pushing the image to dockerhub"
                withCredentials([usernamePassword(credentialsId:"dockerhub",passwordVariable:"dockerhubPass",usernameVariable:"dockerhubUser")]){
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

pipeline {
agent any
stages {

    stage("Clone"){
        steps{
            checkout scm 
            echo 'Code clone in /var/lib/jenkins/workspace'
        }
    }
    stage("Build"){
        steps{
            echo "started docker build image for tag ${env.BUILD_NUMBER}"
            sh "docker build --no-cache -t adkharat/react-currency-exchange-app-fe:${env.BUILD_NUMBER} ./front-end-react-app"
            echo "code build done on tag ${env.BUILD_NUMBER}"
        }
    }
    stage("Test image"){
        steps{
            echo 'Testing Image'
        }
    }
    stage("Push"){
        steps{
            withCredentials([usernamePassword(credentialsId:"docker",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
            sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
            sh "docker push adkharat/react-currency-exchange-app-fe:${env.BUILD_NUMBER}"
            echo 'image pushed'
            }
        }
    }
    stage('Trigger ManifestUpdate - GitOps') {
        steps {
                echo "Triggering updatemanifestjob"
                build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
            }
    }
}
}
pipeline {
agent any
tools {
        // jdk 'jdk17'
        gradle 'gradle8-7' //name of  gradle tool configured in Jenkins tool section
}
stages {
    stage('Gradle version') {
        steps {
            sh 'pwd'
            sh 'ls -la'
            echo 'Gradle version is '
            sh 'gradle --version'
        }
    }
    stage('CleanWorkspace') {
        steps {
            cleanWs()
            echo 'Workspace cleaned /var/lib/jenkins/workspace'
        }
    }
    stage("Clone"){
        steps{
            checkout scm 
            echo 'Code clone in Workspace /var/lib/jenkins/workspace'
        }
    }
    stage("Gradle build"){
        steps{
            echo 'Generating the Gradle build in build/lib/ folder' //https://tomgregory.com/gradle/gradle-assemble-task-essentials/
            sh "pwd"
            dir('first_spring_boot_to_RDS') {
                sh "pwd"
                sh "./gradlew assemble"  //Make sure gradle is configured/installed in tool section of Jenkins
            }
            sh "pwd"
            dir('second_spring_boot_to_RDS') {
                sh "pwd"
                sh "./gradlew assemble"  //Make sure gradle is configured/installed in tool section of Jenkins
            }
            sh "pwd"
        }
    }
    stage("Docker Build"){
        steps{
            echo "started docker build image for tag ${env.BUILD_NUMBER}"
            sh "docker build --no-cache -t adkharat/react-currency-exchange-app-fe:${env.BUILD_NUMBER} ./front-end-react-app"
            sh "docker build --no-cache -t adkharat/first_spring_boot_to_rds_1:${env.BUILD_NUMBER} ./first_spring_boot_to_RDS"
            sh "docker build --no-cache -t adkharat/second_spring_boot_to_rds_1:${env.BUILD_NUMBER} ./second_spring_boot_to_RDS"

            echo "code build done on tag ${env.BUILD_NUMBER}"
        }
    }
    stage("Test image"){
        steps{
            echo 'Testing Image'
        }
    }
    stage("Docker Push"){
        steps{
            withCredentials([usernamePassword(credentialsId:"docker",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
            sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
            sh "docker push adkharat/react-currency-exchange-app-fe:${env.BUILD_NUMBER}"
            sh "docker push adkharat/first_spring_boot_to_rds_1:${env.BUILD_NUMBER}"
            sh "docker push adkharat/second_spring_boot_to_rds_1:${env.BUILD_NUMBER}"
            echo 'image pushed'
            }
        }
    }
    stage("Docker Clean up"){
        steps{
            echo 'Clean up started'
            sh "docker rmi adkharat/react-currency-exchange-app-fe:${env.BUILD_NUMBER}"
            sh "docker rmi adkharat/first_spring_boot_to_rds_1:${env.BUILD_NUMBER}"
            sh "docker rmi adkharat/second_spring_boot_to_rds_1:${env.BUILD_NUMBER}"
            echo 'Cleanup Done'
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
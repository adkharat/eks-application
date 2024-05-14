pipeline {

    agent any

    environment {
        EMAIL_TO = 'ajay.dilip@kickdrumtech.com'
        EMAIL_CC = 'ajaykharat17@gmail.com'
    }

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
        // stage("Gradle build"){
        //     steps{
        //         echo 'Generating the Gradle build in build/lib/ folder' //https://tomgregory.com/gradle/gradle-assemble-task-essentials/
        //         sh "pwd"
        //         dir('first_spring_boot_to_RDS') {
        //             sh "pwd"
        //             sh "./gradlew assemble"  //Make sure gradle is configured/installed in tool section of Jenkins
        //         }
        //         sh "pwd"
        //         dir('second_spring_boot_to_RDS') {
        //             sh "pwd"
        //             sh "./gradlew assemble"  //Make sure gradle is configured/installed in tool section of Jenkins              
        //         }
        //         sh "pwd"
        //     }
        // }
        // stage("Static code analysis_Sonarqube"){
        //     steps{
        //         dir('first_spring_boot_to_RDS') {
        //             script{
        //                 //Gradle build 
        //                 withSonarQubeEnv(credentialsId: 'sonar') {
        //                         sh './gradlew sonar' //Make sure gradle plugin ias added in build.gradle file
        //                         echo "Sonar hosta URL is ${env.SONAR_HOST_URL}"
        //                         echo "Sonar auth token is ${env.SONAR_AUTH_TOKEN}"
        //                 }

        //                 //quality gate status check
        //                 timeout(time: 10, unit: 'MINUTES') {
        //                 def qg = waitForQualityGate()
        //                 if (qg.status != 'OK') {
        //                     error "Pipeline aborted due to quality gate failure: ${qg.status}"
        //                 }
        //                 }
        //             }
        //         }
        //         dir('second_spring_boot_to_RDS') {
        //             script{
        //                 //Gradle build 
        //                 withSonarQubeEnv(credentialsId: 'sonar') {
        //                         sh './gradlew sonar' //Make sure gradle plugin ias added in build.gradle file
        //                 }
        //                 //quality gate status check
        //                 timeout(time: 10, unit: 'MINUTES') {
        //                 def qg = waitForQualityGate()
        //                 if (qg.status != 'OK') {
        //                     error "Pipeline aborted due to quality gate failure: ${qg.status}"
        //                 }
        //                 }
        //             }                
        //         }
        //     }
        // }
        // stage ('OWASP Dependency-Check Vulnerabilities') {
        //     steps {
        //         dependencyCheck additionalArguments: ''' 
        //             -o "./" 
        //             -s "./"
        //             -f "ALL" 
        //             --prettyPrint''', odcInstallation: 'DP-check'

        //         dependencyCheckPublisher pattern: 'dependency-check-report.xml'
        //     }
        // } 
        stage("Docker Build"){
            steps{
                echo "started docker build image for tag ${env.BUILD_NUMBER}"
                sh "docker build --no-cache -t adkharat/react-currency-exchange-app-fe:${env.BUILD_NUMBER} ./front-end-react-app"
                sh "pwd"
                sh "ls -la"
                // sh "docker build --no-cache -t adkharat/first_spring_boot_to_rds_1:${env.BUILD_NUMBER} ./first_spring_boot_to_RDS"
                // sh "docker build --no-cache -t adkharat/second_spring_boot_to_rds_1:${env.BUILD_NUMBER} ./second_spring_boot_to_RDS"

                echo "code build done on tag ${env.BUILD_NUMBER}"
            }
        }
        stage("Image vulnerability"){
            steps{
                echo "Started : Checking Image vulnerability"
                sh "trivy image adkharat/react-currency-exchange-app-fe:${env.BUILD_NUMBER} > scanning_frontend.txt"
                // sh "trivy image adkharat/first_spring_boot_to_rds_1:${env.BUILD_NUMBER} > scanning_backend_1.txt"
                // sh "trivy image adkharat/second_spring_boot_to_rds_1:${env.BUILD_NUMBER} > scanning_backend_2.txt"
                echo "Done : Checking Image vulnerability"
            }
        }
        // stage("Docker Push"){
            // steps{
            //     withCredentials([usernamePassword(credentialsId:"docker",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
            //     sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
            //     sh "docker push adkharat/react-currency-exchange-app-fe:${env.BUILD_NUMBER}"
            //     sh "docker push adkharat/first_spring_boot_to_rds_1:${env.BUILD_NUMBER}"
            //     sh "docker push adkharat/second_spring_boot_to_rds_1:${env.BUILD_NUMBER}"
            //     echo 'image pushed'
            //     }
            // }
        // }
        // stage("Docker Clean up"){
        //     steps{
        //         echo 'Clean up started'
        //         sh "docker rmi adkharat/react-currency-exchange-app-fe:${env.BUILD_NUMBER}"
        //         sh "docker rmi adkharat/first_spring_boot_to_rds_1:${env.BUILD_NUMBER}"
        //         sh "docker rmi adkharat/second_spring_boot_to_rds_1:${env.BUILD_NUMBER}"
        //         echo 'Cleanup Done'
        //     }
        // }
        // stage('Trigger ManifestUpdate - GitOps') {
        //     steps {
        //             echo "Triggering updatemanifestjob"
        //             build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
        //     }
        // }
    }

    post {
        failure {
                mail to: "${EMAIL_TO}",
                    cc : "${EMAIL_CC}",
                subject: "FAILED: Build ${env.JOB_NAME}", 
                body: "Build failed ${env.JOB_NAME} build no: ${env.BUILD_NUMBER}.\n\nView the log at:\n ${env.BUILD_URL}\n\nBlue Ocean:\n${env.RUN_DISPLAY_URL}"
        }
        
        success {
                emailext attachmentsPattern: '*.txt',
                    // mail to: "${EMAIL_TO}",
                    //     cc : "${EMAIL_CC}",
                    to: "${EMAIL_TO}",
                    body: "Build Successful ${env.JOB_NAME} build no: ${env.BUILD_NUMBER}\n\nView the log at:\n ${env.BUILD_URL}\n\nBlue Ocean:\n${env.RUN_DISPLAY_URL}",
                    subject: "SUCCESSFUL: Build ${env.JOB_NAME}"
        }
            
        aborted {
                mail to: "${EMAIL_TO}",
                    cc : "${EMAIL_CC}",
                    subject: "ABORTED: Build ${env.JOB_NAME}", 
                    body: "Build was aborted ${env.JOB_NAME} build no: ${env.BUILD_NUMBER}\n\nView the log at:\n ${env.BUILD_URL}\n\nBlue Ocean:\n${env.RUN_DISPLAY_URL}"
        }
    }
}
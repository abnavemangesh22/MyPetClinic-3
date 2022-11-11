pipeline{
    agent any
    tools{
        maven 'M3'
    }
    stages{
        stage('Code Commit'){
            steps{
                git url: 'https://github.com/abnavemangesh22/MyNewPetClinic-2.git'
            }
        }
        stage('Code Secrets'){
            steps{
                sh 'rm -rf trufflehog-1'
                sh 'docker run --rm gesellix/trufflehog --json https://github.com/abnavemangesh22/MyNewPetClinic-2.git > trufflehog-1'
                sh 'cat trufflehog-1'
            }
        }
        stage('Code Compilation'){
            steps{
                sh 'mvn clean test'
            }
        }
        stage('Static code Analysis'){
           steps{
               script{
                    withSonarQubeEnv('sonarsrv'){
                         sh "mvn sonar:sonar"
                     }
                }
                
            }
        }
        stage('Source Code Composition Analysis'){
            steps{
                sh 'mvn clean install -Powasp-dependency-check'
            }
        }
        stage('code Packaging'){
            steps{
                sh 'mvn package -DskipTests'
            }
        }
        stage('Uploading the Repo to the Nexus'){
            steps{
                script{
                    nexusArtifactUploader artifacts: [[artifactId: 'spring-petclinic', classifier: '', file: 'target/spring-petclinic-1.5.2.jar', type: 'jar']], credentialsId: 'nexusrpo', groupId: 'org.springframework.samples', nexusUrl: '192.168.11.160:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'pipeline-repo', version: '1.5.2'
                }
            }
        }
        stage('Image Build'){
            steps{
                sh 'docker build -t mangeshabnave/mycentralbank-3:latest .'
            }
        }
        stage('upload the image'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'password', usernameVariable: 'username')]) {
                        sh "docker login -u ${env.username} -p ${env.password}"
                        sh "docker push mangeshabnave/mycentralbank-3:latest"
                      }
            }
        }
        stage('Deploy the application on Remote'){
            steps{
                sh "sudo ssh -o StrictHostKeyChecking=no root@192.168.15.190 'docker stop webapp'"
                sh "sudo ssh -o StrictHostKeyChecking=no root@192.168.15.190 'docker pull mangeshabnave/mycentralbank-3:latest'"
                sh "sudo ssh -o StrictHostKeyChecking=no root@192.168.15.190 'docker run -dit --name webpp -p 45000:8080 mangeshabnave/mycentralbank-3:latest'"
            }
        }
    }
    
    
}

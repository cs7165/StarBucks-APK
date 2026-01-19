pipeline {
    agent any

    tools {
        jdk 'jdk'
        nodejs 'node17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME   = "sagarbarve/starbucks"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/cs7165/StarBucks-APK.git'
            }
        }

        // stage('SonarQube Analysis') {
        //     steps {
        //         withSonarQubeEnv('SonarQube') {
        //             sh """
        //             ${SCANNER_HOME}/bin/sonar-scanner \
        //             -Dsonar.projectName=starbucks \
        //             -Dsonar.projectKey=starbucks \
        //             -Dsonar.projectBaseDir=. \
        //             -Dsonar.sources=.
        //             """
        //         }
        //     }
        // }

        // stage('Quality Gate') {
        //     steps {
        //         waitForQualityGate(
        //             abortPipeline: false,
        //             credentialsId: 'SonarQube'
        //         )
        //     }
        // }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withDockerRegistry(
                    credentialsId: 'docker',
                    url: 'https://index.docker.io/v1/'
                ) {
                    sh '''
                    docker build -t starbucks .
                    docker tag starbucks sagarbarve/starbucks:latest
                    docker push sagarbarve/starbucks:latest
                    '''
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image sagarbarve/starbucks:latest > trivyimage.txt'
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                docker rm -f starbucks || true
                docker run -d --name starbucks -p 3000:3000 sagarbarve/starbucks:latest
                '''
            }
        }
    }

    post {
        always {
            script {
                def buildStatus = currentBuild.currentResult
                def buildUser = currentBuild.getBuildCauses(
                    'hudson.model.Cause$UserIdCause'
                )[0]?.userId ?: 'Github User'

                emailext(
                    subject: "Pipeline ${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                    <h3>Jenkins Starbucks CI/CD Pipeline Status</h3>
                    <p><b>Project:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>Build Status:</b> ${buildStatus}</p>
                    <p><b>Started By:</b> ${buildUser}</p>
                    <p><b>Build URL:</b>
                    <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                    """,
                    to: 'sagarbarve1416@gmail.com',
                    from: 'sagarbarve1416@gmail.com',
                    replyTo: 'sagarbarve1416@gmail.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
                )
            }
        }
    }
}

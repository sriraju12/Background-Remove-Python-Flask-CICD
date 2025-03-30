pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = "sriraju12/background-remover-python-app:${BUILD_NUMBER}"
    }
    stages {
        stage ("Clean workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/sriraju12/Background-Remove-Python-Flask-CICD.git'
            }
        }
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=background-remover-python-app \
                    -Dsonar.projectKey=background-remover-python-app '''
                }
            }
        }
        
        stage('OWASP FS Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-CHECK'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ("Trivy File Scan") {
            steps {
                sh "/opt/homebrew/bin/trivy fs --format table -o trivy-fs-report.html ."
            }
        }
       stage('Build and Push Docker Image') {
         environment {
            REGISTRY_CREDENTIALS = credentials('dockerhub-token')
      }
      steps {
        script {
            sh 'docker context use default'  
            sh "docker build -t ${DOCKER_IMAGE} ."
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "dockerhub-token") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Docker Image Scan') {
       steps {
           sh "/opt/homebrew/bin/trivy image --format table -o trivy-image-report.html ${DOCKER_IMAGE}"
           }
        }


    }
     post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'rajukrishnamsetty9@gmail.com',                                
            attachmentsPattern: 'trivy-fs-report.html,trivy-image-report.html'
        }
    }   
}

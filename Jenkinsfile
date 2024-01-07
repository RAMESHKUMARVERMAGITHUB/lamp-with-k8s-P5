pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'master', url: 'https://github.com/rameshkumarvermagithub/lamp-with-k8s-P5.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=lamp-with-k8s-p5 \
                    -Dsonar.projectKey=lamp-with-k8s-p5'''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
        // stage('Install Dependencies') {
        //     steps {
        //         sh "npm install"
        //     }
        // }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                    dir('lamp-app'){
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t lamp-with-k8s-p5 ."
                       sh "docker tag lamp-with-k8s-p5 rameshkumarverma/lamp-with-k8s-p5:latest"
                       sh "docker push rameshkumarverma/lamp-with-k8s-p5:latest"
                        }
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image rameshkumarverma/lamp-with-k8s-p5:latest > trivyimage.txt"
            }
        }
        // stage("deploy_docker"){
        //     steps{
        //         sh "docker stop flaskapp || true"  // Stop the container if it's running, ignore errors
        //         sh "docker rm flaskapp || true" 
        //         sh "docker run -d --name amazon -p 4000:3000 rameshkumarverma/flaskapp:latest"
        //     }
        // }
        stage('deploy_docker'){
            steps{
                script{
                    dir('lamp-app'){
                        sh 'docker-compose up -d'
                    }
                }
            }
        }
      // stage('Deploy to Kubernetes') {
      //       steps {
      //           script {
      //               dir('kubernetes') {
      //                   withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
      //                       sh 'kubectl apply -f mysql.yml'
      //                       sh 'kubectl apply -f persistentvolumeclaim.yaml'
      //                       sh 'kubectl apply  -f app.yml'
      //                   }
      //               }
      //           }
      //       }
      //   }

    }
}

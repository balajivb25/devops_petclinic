pipeline {
    agent any

    tools {
        maven 'Maven3.9.9'    // Name from Global Tool Config
        jdk 'JDK21'           // Name from Global Tool Config
    }

    environment {
        DEPLOY_URL = 'http://localhost:9090'
        TOMCAT_CREDS = 'tomcat10-admin'
        GIT_REPO = 'https://github.com/balajivb25/devops_petclinic.git'
        GIT_BRANCH = 'main'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${env.GIT_BRANCH}",
                    credentialsId: 'github-https',
                    url: "${env.GIT_REPO}"
            }
        }

        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
                sh 'mvn -B clean package -DskipTests'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
		
        stage('Find WAR Files') {
            steps {
                script {
                    def files = findFiles(glob: '**/*.war')
                    files.each { f ->
                    echo "Found WAR file: ${f.path}"
                    }
                }
            }
        }
		
        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def wars = findFiles(glob: '**/target/*.war')
                    for (w in wars) {
                        def appName = w.name.replace('.war','')
                        echo "Deploying ${appName} → ${DEPLOY_URL}"
                        deploy adapters: [
                            tomcat9(
                                credentialsId: "${env.TOMCAT_CREDS}",
                                url: "${env.DEPLOY_URL}"
                            )
                        ], contextPath: appName, war: w.path
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build & Deployment successful!'
        }
        failure {
            emailext(
                to: "balajiv.b25@gmail.com",
                subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                The Jenkins build has failed.
                Job: ${env.JOB_NAME}
                Build: #${env.BUILD_NUMBER}
                URL: ${env.BUILD_URL}
                """
            )
        }
    }
}

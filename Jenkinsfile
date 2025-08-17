pipeline {
    agent any

    tools {
        maven 'Maven3.9.9'    // From Global Tool Config
        jdk 'JDK21'
    }

    environment {
        DEPLOY_URL   = 'http://localhost:9090'
        TOMCAT_CREDS = 'tomcat10-admin'
        GIT_REPO     = 'https://github.com/balajivb25/devops_petclinic.git'
        GIT_BRANCH   = 'main'
    }

    stages {

        stage('Init') {
            steps {
                wrap([$class: 'BuildUser']) {
                    echo "Build triggered by: ${BUILD_USER}"
                    echo "User ID: ${BUILD_USER_ID}"
                    //echo "Full Name: ${BUILD_USER_FULL_NAME}"
                    //echo "Email: ${BUILD_USER_EMAIL}"
                    //script {
                    //currentBuild.displayName = "#${env.BUILD_NUMBER} - ${env.BUILD_USER}"
                    //currentBuild.description = "Triggered by ${BUILD_USER} on commit ${GIT_COMMIT[0..6]}"
                }
                }
                
            }
        
        stage('Checkout') {
                steps {
                    checkout scm  // automatically uses the repo configured in Jenkins job
                    script {
                        def commit = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                        currentBuild.displayName = "#${env.BUILD_NUMBER} - ${BUILD_USER} (${commit})"
                    }
                }
            }

        /*stage('Checkout') {
            steps {
                git branch: "${env.GIT_BRANCH}",
                    credentialsId: 'github-https',
                    url: "${env.GIT_REPO}"

                script {
                    //def author = sh(returnStdout: true, script: "git log -1 --pretty=format:'%an'").trim()
                    //def commitHash = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    currentBuild.displayName = "#${env.BUILD_NUMBER} - ${env.GIT_BRANCH} - ${BUILD_USER_ID}"
                    //currentBuild.description = "Commit ${commitHash} by ${author} (Triggered by ${BUILD_USER})"
                    currentBuild.description = "Triggered by ${BUILD_USER} on commit ${GIT_COMMIT[0..6]}"
                }
            }
        }*/

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
                deploy adapters: 
                    [
                    tomcat9
                                  (
                                      credentialsId: 'tomcat10-admin',
                                      url: 'http://localhost:9090'
                                      //credentialsId: "${env.TOMCAT_CREDS}",
                                      path: '', // Context path, leave empty for ROOT 
                                      //url: "${env.DEPLOY_URL}"
                                  ) 
                    ], contextPath: 'petclinic', war: '**/target/*.war'
                /*script {
                    def wars = findFiles(glob: '**/target/*.war')
                    for (w in wars) {
                        def appName = w.name.replace('.war','')
                        echo "Deploying ${appName} → ${DEPLOY_URL}"
                        deploy adapters: [
                            tomcat10(
                                credentialsId: "${env.TOMCAT_CREDS}",
                                url: "${env.DEPLOY_URL}"
                            )
                        ], contextPath: appName, war: w.path
                    }
                }*/
                
            }
        }
    }

    post {
        success {
            echo '✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}'
            /*emailext(
                to: "${BUILD_USER_EMAIL}",
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Hi ${BUILD_USER_FULL_NAME},

                Your build succeeded.

                Job: ${env.JOB_NAME}
                Build: #${env.BUILD_NUMBER}
                URL: ${env.BUILD_URL}
                """
            )*/
        }
        failure {
            echo '❌ Build failed! ${env.JOB_NAME} #${env.BUILD_NUMBER}'
            /*emailext(
                to: "balajiv.b25@gmail.com",
                subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                The Jenkins build has failed.

                Job: ${env.JOB_NAME}
                Build: #${env.BUILD_NUMBER}
                URL: ${env.BUILD_URL}
                """
            )*/
        }
    }
}

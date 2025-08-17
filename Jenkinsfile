pipeline {
    agent any

    tools {
        maven 'Maven3.9.9' // From Global Tool Config
        jdk 'JDK21'
    }

    environment {
        DEPLOY_URL = 'http://localhost:9090'
        TOMCAT_CREDS = 'tomcat10-admin'
        //GIT_REPO = 'https://github.com/balajivb25/devops_petclinic.git'
        //GIT_BRANCH = 'main'
    }

    stages {

        stage('Init & CheckOut') {
            steps {
                wrap([$class: 'BuildUser']) {
                    echo "Build triggered by: ${BUILD_USER}"
                    echo "User ID: ${BUILD_USER_ID}"
                    checkout scm // automatically uses the repo configured in Jenkins job
                    script {
                        // Short commit hash
                        def commit = bat(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                        // Committer email
                        //def committerEmail = bat(returnStdout: true, script: 'git --no-pager show -s --format=%ce').trim() 
                        //echo "COMMITTER_EMAIL: ${COMMITTER_EMAIL}" 
                        // Optional: set build display name
                        currentBuild.displayName = "#${env.BUILD_NUMBER} by ${BUILD_USER} (CommitID ${commit})"
                        currentBuild.description = "Triggered by ${BUILD_USER} on commit ${commit}" //and GitHub User: ${COMMITTER_EMAIL}"
                    }
				}
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
                        files.each {
                        f->
                        echo "Found WAR file: ${f.path}"
                    }
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: '**/target/*.war',
                fingerprint: true
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                deploy adapters:
                [
                    tomcat9
                    (
                        //credentialsId: 'tomcat10-admin',
                        //url: 'http://localhost:9090'
                        credentialsId: "${env.TOMCAT_CREDS}",
                        path: '', // Context path, leave empty for ROOT
                        url: "${env.DEPLOY_URL}"
                    )
                ],
                contextPath: 'petclinic',
                war: '**/target/*.war'
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

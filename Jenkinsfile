pipeline {
    agent any
    tools {
        maven 'Maven3.9.9'   // must match Global Tool Config
        jdk 'JDK21'          // must match Global Tool Config
    }
    stages {
        stage('Build') { 
            steps {
                sh 'java -version'
                sh 'mvn -version'
                //sh 'mvn -B -DskipTests clean package'
                sh 'mvn clean install'
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

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def wars = findFiles(glob: '**/target/*.war')
                    for (w in wars) {
                    def appName = w.name.replace('.war','')
                    deploy adapters: [
                        tomcat9(
                            credentialsId: 'tomcat10-admin',
                            url: 'http://localhost:9090'
                        )
                    ], contextPath: appName, war: w.path
                    }
                }
            }
        }
    }
}

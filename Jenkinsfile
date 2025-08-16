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
                sh 'mvn -B -DskipTests clean package'
                //sh 'mvn clean install'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [
                    tomcat9(
                        credentialsId: 'tomcat10-admin',   // Jenkins credential ID
                        path: '',                              // Context path, leave empty for ROOT
                        url: 'http://localhost/:9090'       // Tomcat manager URL
                    )
                ], contextPath: 'petclinic', war: '**/*.war'
            }
        }
    }
}

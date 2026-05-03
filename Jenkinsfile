pipeline{
    agent any 
    tools{
        maven 'Maven-3.9'
        jdk 'JDK-17'
    }
    stages{
        stage('cl'){
            steps{
                git branch : 'main',
                url: 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }
        stage('change config'){
            steps{
                sh 'echo server.port=8081 >> src/main/resources/application.properties'
            }
        }
        stage('compile'){
            steps{
                sh 'mvn clean compile'
            }
        }
        stage('test'){
            steps{
                sh 'mvn test'
            }
        }
        stage('package'){
            steps{
                sh 'mvn package'
            }
        }
        stage('run'){
            steps{
                sh 'nohup java -jar target/*.jar --server.port=8081 > app.log 2>&1 & sleep 300'
            }
        }
    }
}

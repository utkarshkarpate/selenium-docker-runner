pipeline {
    agent any

    stages(){
        stage('Run-Test'){
            steps{
                bat "docker-compose up" //build the jar
            }
        }
        stage('Bring-grid-down'){
            steps{
                bat "docker-compose down"
            }
        }
    }
}
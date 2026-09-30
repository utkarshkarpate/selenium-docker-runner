pipeline {
    agent any

    stages(){
        stage('Start-Grid'){
            steps{
                bat "docker-compose -f seleniumgrid.yaml up -d"
            }
        }
        stage('Run-Test'){
            steps{
                bat "docker-compose -f test-suites.yaml up"
            }
        }

    }
    post {
        always {
            bat "docker-compose -f seleniumgrid.yaml down"
            bat "docker-compose -f test-suites.yaml down"
        }
    }
}
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
            archiveArtifacts artifacts: 'output/flight-reservation/emailable-report.html', followSymlinks: false
            archiveArtifacts artifacts: 'output/vendor-portal/emailable-report.html', followSymlinks: false

            /*with the archive step, we will start viewing the report in jenkins UI as well now*/
        }
    }
}
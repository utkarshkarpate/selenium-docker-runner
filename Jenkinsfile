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
            archiveArtifacts artifacts: 'output/**', followSymlinks: false
            archiveArtifacts artifacts: 'output/**', followSymlinks: false
            /*with the archive step, we will start viewing the report in jenkins UI as well now
            but when we try to open the html link, we will not be able to view the report
            as we were viewign it in our local. to solve that, we will use
                environment:
      - JAVA_OPTS="-DHudson.model.DirectoryBrowserSupport.CSP="

      in our docker compose file for jenkins and restart jenkins*/
        }
    }
}
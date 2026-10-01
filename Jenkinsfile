pipeline {
    agent any
    parameters {
        choice choices: ['chrome', 'firefox'], description: 'select browser', name: 'BROWSER'
    }

    /*this will add the option to select browser when we are building the job in jenkins. we can select the browser and run the test in that browser
    Jenkins gives this syntax by going to the job->pipeline syntax->Generate Declarative Generator
    These paramaters are applicable for all the stages {
    We cannot change it in any other stage now.post {
    THis is the biggest difference between these env variables and the one which we set using envrionment
        }
    }*/

    stages(){
        stage('Start-Grid'){
            steps{
                bat "docker-compose -f seleniumgrid.yaml up --scale ${params.BROWSER}=2 -d" //based of the browser we select, we will scale it using params.BROWSER
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
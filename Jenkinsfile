pipeline {
    agent any
    parameters {
        choice choices: ['chrome', 'firefox'], description: 'select browser', name: 'BROWSER'
        /*
        By default when we run for the first time, jenkins will pick up the first value in the parameters
        That is the reason when we pushed the code to github and did build now, our tests ran in chrome
        From the second time, we will start getting Build With Paramaters option in Jenkins for our job
        We can also paramterise thread count in our case if we want
        */

        /*choice choices: ['vendor-portal', 'flight-reservation'], description: 'select test suite to run', name: 'test-suites'*/


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
                bat "docker-compose -f test-suites.yaml up --pull=always"

                /*--pull=always will ensure that we are pulling the latest image from docker hub always*/

                /*When we run the test with this approach, we will notice that our stage in jenkins
        shows success. When we go to volumes->node-><our job>->output->vendor-portal, we will
        notice that there is a failed testng xml file. We will use that file existence and
        write the status as Success or failed in jenkins*/
                script{
                    // TestNG writes testng-failed.xml in a suite folder when that suite has failures.
                    // dir searches every child folder under output. Exit code 0 means the file was found.
                    def failedReport = bat(returnStatus: true, script: 'dir /s /b output\\testng-failed.xml >nul 2>&1')
                    if (failedReport == 0) {
                        error("TestNG failed xml file exists, marking build as failed")
                    }
                }
            }
        }



    }
    post {
        always {
            bat "docker-compose -f seleniumgrid.yaml down"
            bat "docker-compose -f test-suites.yaml down"
            // Copy the reports to the job root so Archived Artifacts shows the HTML links,
            // not the output folder and its css/js files.
            bat "copy /Y output\\flight-reservation\\emailable-report.html flight-reservation-report.html"
            bat "copy /Y output\\vendor-portal\\emailable-report.html vendor-portal-report.html"
            archiveArtifacts artifacts: 'flight-reservation-report.html, vendor-portal-report.html', followSymlinks: false
            /* Opening these links needs the Jenkins CSP option:
                JAVA_OPTS=-Dhudson.model.DirectoryBrowserSupport.CSP=
               in jenkins-ci-cd/docker-compose.yaml */
        }
    }
}
pipeline {
    agent any
    parameters {
        choice choices: ['chrome', 'firefox'], description: 'select browser', name: 'BROWSER'
        /*
        By default when we run for the first time, jenkins will pick up the first value in the parameters
        That is the reason when we pushed the code to github and did build now, our tests ran in chrome
        From the second time, we will start getting Build With Parameters option in Jenkins for our job
        We can also parameterise thread count in our case if we want
        */

        text(
            name: 'TEST_SUITES',
            defaultValue: '''vendor-portal
flight-reservation''',
            description: 'One suite name per line'
        )
    }

    stages {
        stage('Start-Grid') {
            steps {
                bat "docker-compose -f seleniumgrid.yaml up --scale ${params.BROWSER}=2 -d"
            }
        }
        stage('Run-Test') {
            steps {
                bat "docker-compose -f test-suites.yaml pull"
                script {
                    def suites = params.TEST_SUITES
                        .split(/[\r\n,]+/)
                        .collect { it.trim() }
                        .findAll { it }

                    def failed = []
                    // Run a few containers at a time so they fit the grid (2 nodes x 5 sessions).
                    suites.collate(4).each { batch ->
                        def branches = [:]
                        batch.each { name ->
                            def suite = name
                            branches[suite] = {
                                def status = bat(returnStatus: true, script: """
                                    set BROWSER=${params.BROWSER}
                                    set TEST_SUITE=${suite}
                                    docker-compose -f test-suites.yaml run --rm --name suite-${suite} test
                                """)
                                return status == 0 ? '' : suite
                            }
                        }
                        def results = parallel branches
                        failed.addAll(results.values().findAll { it })
                    }

                    if (failed) {
                        error("Failed suites: ${failed.join(', ')}")
                    }

                    // TestNG writes testng-failed.xml in a suite folder when that suite has failures.
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
            script {
                def suites = params.TEST_SUITES
                    .split(/[\r\n,]+/)
                    .collect { it.trim() }
                    .findAll { it }
                suites.each { suite ->
                    bat "if exist output\\${suite}\\emailable-report.html copy /Y output\\${suite}\\emailable-report.html ${suite}-report.html"
                }
            }
            archiveArtifacts artifacts: '*-report.html', allowEmptyArchive: true, followSymlinks: false
            /* Opening these links needs the Jenkins CSP option:
                JAVA_OPTS=-Dhudson.model.DirectoryBrowserSupport.CSP=
               in jenkins-ci-cd/docker-compose.yaml */
        }
    }
}
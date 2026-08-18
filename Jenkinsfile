pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Check Python Code') {
            steps {
                bat '''
                    echo Checking Python source files...
                    python -m compileall -q .
                '''
            }
        }

        stage('Run Unit Tests') {
            steps {
                bat '''
                    echo Starting Triangle API...

                    start /B python stand_up_triangle_api.py > triangle_api.log 2>&1

                    echo Waiting for Triangle API to start...
                    timeout /t 3 /nobreak > nul

                    echo Running pytest...
                    python -m pytest -v --junitxml=test-results.xml
                '''
            }

            post {
                always {
                    junit testResults: 'test-results.xml',
                          allowEmptyResults: true

                    archiveArtifacts artifacts: 'triangle_api.log',
                                     allowEmptyArchive: true

                    bat '''
                        echo Stopping Triangle API...
                        taskkill /F /IM python.exe /T > nul 2>&1 || exit /b 0
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'BUILD SUCCESSFUL: All automated tests passed.'
        }

        failure {
            echo 'BUILD FAILED: Review the console output and Jenkins test results.'
        }

        always {
            echo 'CI build complete.'
        }
    }
}
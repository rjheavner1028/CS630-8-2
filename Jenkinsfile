pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Check Python Code') {
            steps {
                sh '''
                    echo "Checking Python source files..."
                    python3 -m compileall -q .
                '''
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh '''
                    echo "Starting Triangle API..."

                    python3 stand_up_triangle_api.py > triangle_api.log 2>&1 &
                    API_PID=$!

                    cleanup() {
                        echo "Stopping Triangle API..."
                        kill "$API_PID" 2>/dev/null || true
                        wait "$API_PID" 2>/dev/null || true
                    }

                    trap cleanup EXIT

                    echo "Waiting for Triangle API to start..."

                    READY=0

                    for i in 1 2 3 4 5 6 7 8 9 10
                    do
                        if curl -s \
                            "http://localhost:5082/triangle?a=1&b=1&c=1" \
                            > /dev/null
                        then
                            READY=1
                            break
                        fi

                        sleep 1
                    done

                    if [ "$READY" -ne 1 ]; then
                        echo "ERROR: Triangle API failed to start."
                        echo "API log:"
                        cat triangle_api.log
                        exit 1
                    fi

                    echo "Triangle API is running."
                    echo "Running pytest..."

                    python3 -m pytest -v --junitxml=test-results.xml
                '''
            }

            post {
                always {
                    junit testResults: 'test-results.xml',
                          allowEmptyResults: true

                    archiveArtifacts artifacts: 'triangle_api.log',
                                     allowEmptyArchive: true
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
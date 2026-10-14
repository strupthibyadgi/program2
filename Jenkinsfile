pipeline {
    agent any

    environment {
        IMAGE_NAME = 'program2-flask'
        DEV_CONTAINER = 'program2-dev'
        TEST_CONTAINER = 'program2-test'
    }

    stages {
        stage('Build development image') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'docker build --pull -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
                    } else {
                        bat 'docker build --pull -t %IMAGE_NAME%:%BUILD_NUMBER% .'
                    }
                }
            }
        }

        stage('Run development container') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            docker rm -f ${DEV_CONTAINER} >/dev/null 2>&1 || true
                            docker run -d --name ${DEV_CONTAINER} -p 5001:5000 ${IMAGE_NAME}:${BUILD_NUMBER}
                            for attempt in $(seq 1 30); do
                                if curl --fail --silent http://localhost:5001/health; then
                                    echo
                                    exit 0
                                fi
                                sleep 2
                            done
                            docker logs ${DEV_CONTAINER}
                            exit 1
                        '''
                    } else {
                        bat '''
                            @echo off
                            docker rm -f %DEV_CONTAINER% >nul 2>&1
                            docker run -d --name %DEV_CONTAINER% -p 5001:5000 %IMAGE_NAME%:%BUILD_NUMBER%
                            if errorlevel 1 exit /b 1
                            "%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -ExecutionPolicy Bypass -Command "$ok=$false; for ($i=0; $i -lt 30; $i++) { try { $r=Invoke-WebRequest -UseBasicParsing http://localhost:5001/health -TimeoutSec 2; if ($r.StatusCode -eq 200) { $ok=$true; break } } catch {}; Start-Sleep -Seconds 2 }; if (-not $ok) { docker logs %DEV_CONTAINER%; exit 1 }"
                        '''
                    }
                }
            }
        }

        stage('Run test container') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            docker rm -f ${TEST_CONTAINER} >/dev/null 2>&1 || true
                            docker run -d --name ${TEST_CONTAINER} -p 5002:5000 ${IMAGE_NAME}:${BUILD_NUMBER}
                            for attempt in $(seq 1 30); do
                                if curl --fail --silent http://localhost:5002/health; then
                                    echo
                                    curl --fail --silent http://localhost:5002/ | grep -q 'DevOps app is running'
                                    exit 0
                                fi
                                sleep 2
                            done
                            docker logs ${TEST_CONTAINER}
                            exit 1
                        '''
                    } else {
                        bat '''
                            @echo off
                            docker rm -f %TEST_CONTAINER% >nul 2>&1
                            docker run -d --name %TEST_CONTAINER% -p 5002:5000 %IMAGE_NAME%:%BUILD_NUMBER%
                            if errorlevel 1 exit /b 1
                            "%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -ExecutionPolicy Bypass -Command "$ok=$false; for ($i=0; $i -lt 30; $i++) { try { $r=Invoke-WebRequest -UseBasicParsing http://localhost:5002/ -TimeoutSec 2; if ($r.StatusCode -eq 200 -and $r.Content -like '*DevOps app is running*') { $ok=$true; break } } catch {}; Start-Sleep -Seconds 2 }; if (-not $ok) { docker logs %TEST_CONTAINER%; exit 1 }"
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                if (isUnix()) {
                    sh '''
                        docker rm -f ${DEV_CONTAINER} ${TEST_CONTAINER} >/dev/null 2>&1 || true
                        docker image rm ${IMAGE_NAME}:${BUILD_NUMBER} >/dev/null 2>&1 || true
                    '''
                } else {
                    bat '''
                        @echo off
                        docker rm -f %DEV_CONTAINER% %TEST_CONTAINER% >nul 2>&1
                        docker image rm %IMAGE_NAME%:%BUILD_NUMBER% >nul 2>&1
                        exit /b 0
                    '''
                }
            }
        }
    }
}

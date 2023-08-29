pipeline {
    agent none
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:3.11.5-alpine3.18'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Manual Approval') {
            input {
                message "Lanjutkan ke tahap Deploy?"
                ok "Proceed"
                submitter "user1,user2" // Ganti dengan daftar pengguna yang diizinkan untuk memberikan persetujuan
                parameters {
                    booleanParam(name: 'APPROVE_DEPLOY', defaultValue: false, description: 'Apakah Anda ingin melanjutkan ke tahap Deploy?')
                }
            }
            steps {
                script {
                    if (params.APPROVE_DEPLOY == true) {
                        echo "Pengguna telah menyetujui untuk melanjutkan ke tahap Deploy."
                    } else {
                        error "Pengguna telah memilih untuk menghentikan eksekusi pipeline."
                    }
                }
            }
        }
        stage('Deploy') { 
            agent any
            environment { 
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                dir(path: env.BUILD_ID) { 
                    unstash(name: 'compiled-results') 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'" 
                }
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
                }
                always {
                    timeout(time: 1, unit: 'MINUTES') {
                        script {
                            currentBuild.result = 'SUCCESS'
                            currentBuild.displayName = "#${env.BUILD_NUMBER} - ${env.JOB_NAME} - ${env.BUILD_ID}"
                            currentBuild.description = "Auto-aborted after 1 minute"
                            error("Pipeline aborted after 1 minute")
                        }
                    }
                }
            }
        }
    }
}

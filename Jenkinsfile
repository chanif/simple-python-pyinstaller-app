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
            agent any
            steps {
                input message: 'Lanjutkan ke tahap Deploy?', ok: 'Proceed'
            }
        }
        stage('Deploy') { 
            agent any
            environment { 
                VOLUME = '$(pwd)/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python2'
            }
            steps {
                script {
                    // Menjalankan kontainer dalam mode latar belakang
                    def CONTAINER_ID = sh(script: "docker run -d -v ${VOLUME} ${IMAGE} sleep 60", returnStdout: true).trim()
                    
                    // Menampilkan status kontainer yang berjalan
                    sh "docker ps"
                    
                    // Menunggu selama 1 menit
                    sleep(time: 60, unit: 'SECONDS')
                    
                    // Menghentikan kontainer setelah 1 menit
                    sh "docker stop ${CONTAINER_ID}"
                    
                    // Menampilkan status kontainer setelah dihentikan
                    sh "docker ps -a"
                    
                    // Menghapus kontainer (opsional)
                    sh "docker rm ${CONTAINER_ID}"
                }
            }
        }
    }
}

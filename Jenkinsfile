pipeline {
    agent none 
    stages {
        stage('Build') { 
            agent {
                docker {
                    image 'python:2-alpine' 
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py' 
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
            // 1. 改变 agent 为 any，不在阶段头部进入容器
            agent any 
            
            steps {
                // 2. 使用标准的宿主机命令直接调度 Docker
                // 注意：在 Docker 容器内的 Jenkins 需要确保挂载了 /var/run/docker.sock
                sh 'docker run --rm -v ${WORKSPACE}:/src cdrx/pyinstaller-linux:python2 "pyinstaller --onefile sources/add2vals.py"'
            }
            
            post {
                success {
                    // 3. 此时产物会正常生成在当前工作区的 dist 目录中
                    archiveArtifacts 'dist/add2vals'
                }
            }
        }
    }
}

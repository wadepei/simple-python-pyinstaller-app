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
                script {
                    // 1. 动态获取当前运行的 Jenkins 容器 ID
                    def jenkins_container_id = sh(script: "hostname", returnStdout: true).trim()
                    
                    // 2. 核心修改：使用 宿主机宿导向 挂载
                    // 把 Jenkins 容器内的真实 ${WORKSPACE} 目录，挂载到打包容器认准的 /src 目录下
                    sh "docker run --rm --volumes-from ${jenkins_container_id} -v ${WORKSPACE}:/src cdrx/pyinstaller-linux:python2 'pyinstaller --onefile sources/add2vals.py'"
                }
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

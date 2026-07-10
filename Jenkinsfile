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
                    // 1. 启动一个后台运行的 pyinstaller 容器（让其暂不退出）
                    // 使用 --entrypoint tail -f /dev/null 防止其执行原本有冲突的入口脚本
                    sh 'docker run -d --name pyinstaller_build --entrypoint tail cdrx/pyinstaller-linux:python2 -f /dev/null'
                    
                    try {
                        // 2. 将 Jenkins 当前工作区的所有代码，直接“推”送到容器内的 /src 目录下
                        sh 'docker cp . pyinstaller_build:/src'
                        
                        // 3. 在容器内部执行打包命令（此时文件绝对存在于容器的 /src 中）
                        // 注意：这里我们手动调用镜像原始的 /entrypoint.sh 来确保环境正常
                        sh 'docker exec -w /src pyinstaller_build /entrypoint.sh "pyinstaller --onefile sources/add2vals.py"'
                        
                        // 4. 将容器内打包好的产物（dist目录），“拉”回到 Jenkins 的工作区
                        sh 'docker cp pyinstaller_build:/src/dist .'
                    } finally {
                        // 5. 无论打包成功还是失败，最后都强行清理掉这个临时容器
                        sh 'docker rm -f pyinstaller_build'
                    }
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

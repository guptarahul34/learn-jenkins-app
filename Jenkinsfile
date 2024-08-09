pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -l
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -l
                    echo ${GIT_COMMIT:0:8}
                '''
            }
        }

        stage("Test") {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }

            steps {
                echo "Test Stage"
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }
        }
    }
}
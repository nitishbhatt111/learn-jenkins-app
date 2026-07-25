pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
                sh '''
                    ls -la 
                    node -v
                    npm -v
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
    }
}

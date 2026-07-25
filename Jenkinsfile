pipeline {
    agent any

    stages {
        // stage('Build') {
        //         agent {
        //             docker {
        //                 image 'node:18-alpine'
        //                 reuseNode true
        //             }
        //         }
        //     steps {
        //         sh '''
        //             ls -la 
        //             node -v
        //             npm -v
        //             npm ci
        //             npm run build
        //             ls -la
        //         '''
        //     }
        // }
        stage('Test'){
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Running tests..."
                    test -f build/index.html
                    cat build/index.html
                    npm test
                '''
            }
        }
        stage('E2E'){
            agent{
                docker {
                    image 'docker pull mcr.microsoft.com/playwright:v1.62.0-noble'
                    reuseNode true
                }
            }
            steps {
                // sh '''
                //     echo "Running tests..."
                //     test -f build/index.html
                //     cat build/index.html
                //     npm test
                // '''
                sh '''
                    npm install serve
                    learn-jenkins-app\node_modules\.bin\serve -s build
                    npx playwright test
                '''
            }
        }
    }
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}

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
         stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.61.0-noble'
                    args '--ipc=host'
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
                    npm ci
                    npm run build
                    npm install -D serve
                    npx serve -s build -l 3000 &
                    sleep 10
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

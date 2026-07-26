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

                stage('Tests'){
                    parallel{
                    stage('Unit Test'){
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
                    post {
                        always {
                            // junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.61.0-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            // junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

                stage ('deploy'){
                    agent{
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            npm install netlify-cli
                            node_modules/.bin/netlify --version
                        '''
                    }
                }

            }
        }
        

    }
    
}

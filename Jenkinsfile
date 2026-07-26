pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID="5fb51209-f118-4e3e-91f4-9778ff983708"
        NETLIFY_AUTH_TOKEN=credentials('netlify-token')
    }

    stages {
        stage('Build') {
                agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
            steps {
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
                            echo "Deploying to Netlify... SITE_ID : $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --prod --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN
                        '''
                    }
                }

            }
        }
        

    }
    
}

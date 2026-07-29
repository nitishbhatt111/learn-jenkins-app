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
                    echo "Checking jenking automatic build..."
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
                stage ('Deploy Staging'){
                    agent{
                        docker {
                            image 'node:18'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            npm install netlify-cli node-jq
                            node_modules/.bin/netlify --version
                            echo "Deploying to Netlify... SITE_ID : $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                            node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                        '''
                    script{
                        env.STAGING_URL = sh(script:"node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                    }
                    }
                }

                
                stage('Staging E2E') {
                agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.61.0-noble'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }

            steps {
                sh '''
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
                stage ('Approval') {
                            steps {
                                timeout(15) {
                                input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                                }
                            }
                        }
                // stage ('Deploy Prod'){
                //     agent{
                //         docker {
                //             image 'node:18'
                //             reuseNode true
                //         }
                //     }
                //     steps{
                //         sh '''
                //             npm install netlify-cli
                //             node_modules/.bin/netlify --version
                //             echo "Deploying to Netlify... SITE_ID : $NETLIFY_SITE_ID"
                //             node_modules/.bin/netlify status
                //             node_modules/.bin/netlify deploy --dir=build --prod --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN
                //         '''
                //     }
                // }

                stage('Deploy Prod') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.61.0-noble'
                            reuseNode true
                        }
                    }

                    environment {
                        CI_ENVIRONMENT_URL="https://astounding-brigadeiros-fc2742.netlify.app"
                    }
                    steps {
                        // sh '''
                        //     npm install serve
                        //     node_modules/.bin/serve -s build &
                        //     sleep 10
                        //     npx playwright test --reporter=html
                        // '''
                        sh '''
                            node -version
                            npm install netlify-cli
                            node_modules/.bin/netlify --version
                            echo "Deploying to Netlify... SITE_ID : $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --prod --site=$NETLIFY_SITE_ID --auth=$NETLIFY_AUTH_TOKEN
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            // junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
           }
        }
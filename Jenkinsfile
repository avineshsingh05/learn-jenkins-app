pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '577c2c4b-f2e4-41c6-88c4-f8885b21147b'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {

        
        stage('Build') {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh ''' 
                   echo 'small change'
                   ls -la
                   node --version
                   npm --version
                   npm ci
                   npm run build
                   ls -la
                '''
            }
        }
        
        stage ('Tests') {
            parallel {
                 stage (' Unit Tests') {
                     agent {
                          docker{
                          image 'node:18-alpine'
                           reuseNode true
                        }
                        }  
                        steps{
                             sh '''
                                test -f build/index.html
                                npm test
                                '''
                            }
                        post{
                            always{
                                junit 'jest-results/junit.xml'
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                            }    
                        }                          
                    }   
                }
        }
        stage ('E2E') {
            agent {
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps{
                sh '''
                npm install serve
                node_modules/.bin/serve -s build &
                sleep 10
                npx playwright test --reporter=html
                '''
            }

            post{
                always{
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
    
        }


        stage ('Deploy staging') {
                    agent {
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            }
                        }

                        environment {
                            CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
                        }
                        steps{
                                sh '''

                                npm install netlify-cli@20.1.1 node-jq
                                node_modules/.bin/netlify --version
                                echo "Deploying to Stagging. Site ID: $NETLIFY_SITE_ID"
                                node_modules/.bin/netlify status
                                node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                                CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)
                                npx playwright test --reporter=html

                                '''
                            }

                        post{
                        always{
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                
                    }       
            
        
            }

        
        stage ('Deploy Prod') {
                 agent {
                      docker{
                           image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                           reuseNode true
                        }
                    }

                    environment {
                        CI_ENVIRONMENT_URL = 'https://silly-pegasus-a8efd4.netlify.app'
                    }
                    steps{
                            sh '''
    
                                node --version
                                npm install netlify-cli@20.1.1
                                node_modules/.bin/netlify --version
                                echo "Deploying to Production. Site ID: $NETLIFY_SITE_ID"
                                node_modules/.bin/netlify status
                                node_modules/.bin/netlify deploy --dir=build --prod                        
                                npx playwright test --reporter=html

                            '''
                        }

                    post{
                    always{
                        junit 'jest-results/junit.xml'
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: ' prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
             
                }       
           
       
        }
    }
}       

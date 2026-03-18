pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '577c2c4b-f2e4-41c6-88c4-f8885b21147b'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
    
        }

        
        stage('Deploy') {
                    agent {
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                            sh ''' 
                                npm install netlify-cli@20.1.1
                                node_modules/.bin/netlify --version
                                echo "Deploying to Production. Site ID: $NETLIFY_SITE_ID"
                                node_modules/.bin/netlify status
                                node_modules/.bin/netlify deploy --dir=build --prod
                            '''
                
                
                    }   
                    
        }
        stage ('Prod E2E') {
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

                            npx playwright test --reporter=html

                            '''
                        }

                    post{
                    always{
                        junit 'jest-results/junit.xml'
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
             
                }       
           
       
        }
    }
}       

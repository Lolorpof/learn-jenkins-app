pipeline {
    agent any

    environment {
        // 'NETLIFY' need to be these exact names
        NETLIFY_SITE_ID = '1283706e-767d-4aa8-918e-467ac0e38873'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token') // the same name as in the token name defined on jenkins web
    }

    stages {
       
        stage ('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo "Build Stage" 
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage ('Unit & E2E Test') {
            parallel {
                stage ('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            echo "Test Stage"
                            # test -f build/index.html
                            npm run test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                } 

                stage ('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            /*
                                running as root is not a good idea.
                                this will make files inaccessible to normal users.
                                also, security related stuff.
                            */
                            // args '-u root:root'
                        }
                    }

                    steps {
                        /*  use '&' at the back to make it run in the background (kinda like async)
                        wait for 10 secs 
                        */
                        sh '''
                            echo "E2E Testing Stage"
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npm run test:e2e
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Dev Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                } 
            }
        }

        stage ('Deploy') {
            agent {
                docker { 
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm i netlify-cli
                    node_modules/.bin/netlify --version
                    echo "deploying to prod. Site Id: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod --no-build
                '''
            }
        }

        stage ('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    /*
                        running as root is not a good idea.
                        this will make files inaccessible to normal users.
                        also, security related stuff.
                    */
                    // args '-u root:root'
                }
            }

            // env var only available in Prod E2E stage
            environment {
                CI_ENVIRONMENT_URL = 'https://superb-basbousa-2087ae.netlify.app'
            }

            steps {
                sh '''
                    echo "Prod E2E Testing Stage"
                    npm run test:e2e
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        
    }
    
    
}
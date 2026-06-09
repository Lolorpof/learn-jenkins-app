pipeline {
    agent any

    environment {
        NAME='duangjun'
    }

    stages {
       /* 
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
        */

        stage ('Test') {
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
        }
    }
    
    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
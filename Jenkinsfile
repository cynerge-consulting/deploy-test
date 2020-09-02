// Node.js Shipyard Template v1.2.0

// Embeddable badge configuration. Under normal circumstances there is no need to change these values
def shipyardBuildBadge = addEmbeddableBadgeConfiguration(id: "shipyard-build", subject: "Shipyard Build")

pipeline {
// Tells Jenkins to run the build on an Agent, if possible. If changed, Make sure Agent node has corresponding label
    agent {
        node {
            label 'build'
        }
    }
// Location for setting global environment variables
    environment {
        EMAIL_RECIPIANTS = 'ljolliff@cynerge.com'
        APP_SOURCE = './src/**/**/**/**.html'
        STATUS_SUCCESS = ''
        JENKINS_URL = "${JENKINS_URL}"
        JOB_NAME = "${JOB_NAME}"
        SONAR_TOKEN = credentials('shipyard-sonarqube')
        SONAR_PROJECT = 'shipyard-project'
        SONAR_SOURCE = 'src'
    }

    stages {
        stage('Dependencies') {
// Install app dependencies using NPM
            agent {
                docker {
                    image 'cynergeconsulting/browser-node-12'
                    alwaysPull true
                }
            }
            steps {
                echo 'Installing...'
                sh 'echo $GIT_BRANCH'
                sh 'npm ci'
            }
        }
        stage('Lint Testing') {
// Lint typescript files for formatting and/or syntax errors
            agent {
                docker {
                    image 'cynergeconsulting/browser-node-12'
                    alwaysPull true
                }
            }
            steps {
                sh 'npm run lint'
            }
        }
        stage('Unit Testing') {
            agent {
                docker {
                    image 'cynergeconsulting/browser-node-12'
                    alwaysPull true
                }
            }
            steps {
                sh 'whoami'
                sh 'npm run test'
            }
        }
        stage('Sonarqube Analysis') {
            agent {
                node {
                    label 'master'
                }
            }

            environment {
                scannerHome = tool 'cynerge-sonarqube'
            }
            steps {
                withSonarQubeEnv('Cynerge Sonarqube') {
                    sh 'npm install typescript'
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.login=$SONAR_TOKEN -Dsonar.projectKey=$SONAR_PROJECT -Dsonar.sources=$SONAR_SOURCE"
                }
            }
        }

        stage('Store NPM Artifact') {
            agent {
                docker {
                    image 'cynergeconsulting/browser-node-12'
                    alwaysPull true
                    args '-u root'
                }
            }
            environment { 
                NPM_USER = credentials('nexus-user')
                NPM_PASS = credentials('nexus-pass')
                NPM_REGISTRY = credentials('nexus-repo')
                NPM_EMAIL = 'npm@site.com'
                NPM_RC_PATH = "${env.WORKSPACE}/.npmrc"
            }
            steps {
                sh 'printenv'
                sh 'whoami'
                sh 'npm install -g npm-cli-login'
                sh 'touch .npmrc'
                sh 'npm-cli-login'
                sh 'cat .npmrc'
                sh "npm publish --registry=$NPM_REGISTRY/"
            }
        }
    }  
    post {
        success {
            script {
                env.STATUS_SUCCESS = 'Job Complete!'
                env.JENKINS_URL = "${JENKINS_URL}"
                env.JOB_NAME = "${JOB_NAME}"
                env.BUILD_STATUS = 'Success'
            sh 'printenv'

            emailext body: '''${SCRIPT, template="email_report.template"}''',
            mimeType: 'text/html',
            subject: 'Build # $BUILD_NUMBER - $BUILD_STATUS!',
            to: "${EMAIL_RECIPIANTS}"

            }
        }
        failure {
            script {
                env.STATUS_SUCCESS = 'Job Complete!'
                env.JENKINS_URL = "${JENKINS_URL}"
                env.JOB_NAME = "${JOB_NAME}"
                env.BUILD_STATUS = 'Failure'
            sh 'printenv'

            emailext body: '''${SCRIPT, template="email_report.template"}''',
            mimeType: 'text/html',
            subject: 'Build # $BUILD_NUMBER - $BUILD_STATUS!',
            to: "${EMAIL_RECIPIANTS}"

            }
        }
    
        cleanup {
            cleanWs()
            script {
                shipyardBuildBadge.setStatus('running')
                try {
                    shipyardBuildBadge.setStatus('passing')
                } 
                catch (Exception err) {
                    shipyardBuildBadge.setStatus('failing')
                    shipyardBuildBadge.setColor('red')
                    error 'Build failed'
                }
            }
        }
    }
}

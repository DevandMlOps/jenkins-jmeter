pipeline {
    agent any

    environment {
        SLACK_CHANNEL = '#appdevops'
        JMETER_HOME = "/opt/jmeter" // Ruta a JMeter
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/DevandMlOps/jenkins-jmeter'
                    env.GIT_AUTHOR_NAME = sh(
                        script: "git log -1 --pretty=format:'%an'",
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Run JMeter Tests') {
            steps {
                script {
                    sh "${env.JMETER_HOME}/bin/jmeter -n -t ${env.WORKSPACE}/flask_app_test_json.jmx -l ${env.WORKSPACE}/results.jtl -e -o ${env.WORKSPACE}/informe_html"
                }
            }
        }

        stage('Publish Performance Report') {
            steps {
                // Asegúrate de que el plugin de Performance esté instalado y configurado
                performanceReport sourceDataFiles: '**/results.jtl', parsers: [[$class: 'JMeterParser']]
            }
        }
    }

    post {
        always {
            script {
                def COLOR_MAP = [
                    'SUCCESS': 'good',
                    'FAILURE': 'danger',
                    'UNSTABLE': 'warning',
                    'ABORTED': 'warning'
                ]
                
                def buildStatus = currentBuild.currentResult
                def errorMessage = ""
                
                if (buildStatus == 'FAILURE') {
                    errorMessage = currentBuild.rawBuild.getLog(1000).join('\n')
                    if (errorMessage.length() > 1000) {
                        errorMessage = errorMessage.take(1000) + "... [mensaje truncado]"
                    }
                }
                
                echo 'Enviando notificación a Slack'
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: COLOR_MAP[buildStatus],
                    message: """*${buildStatus}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}*
                        Branch: ${env.GIT_BRANCH}
                        Commit: ${env.GIT_COMMIT}
                        Author: ${env.GIT_AUTHOR_NAME}
                        More Info at: ${env.BUILD_URL}
                        ${buildStatus == 'FAILURE' ? "\nError Details:\n```${errorMessage}```" : ''}
                    """.stripIndent()
                )
            }
        }
    }
}

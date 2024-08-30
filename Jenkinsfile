pipeline {
    agent any
    environment {
        SLACK_CHANNEL = '#appdevops'
        JMETER_HOME = "/opt/jmeter"
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
                    // Limpiar archivos de resultados anteriores
                    sh "rm -rf ${env.WORKSPACE}/informe_html ${env.WORKSPACE}/results.jtl"
                    // Ejecutar las pruebas de JMeter
                    sh "${env.JMETER_HOME}/bin/jmeter -n -t ${env.WORKSPACE}/flask_app_test_json.jmx -l ${env.WORKSPACE}/results.jtl -e -o ${env.WORKSPACE}/informe_html"
                }
            }
        }
        stage('Publish Performance Report') {
            steps {
                perfReport sourceDataFiles: '**/results.jtl', 
                           compareBuildPrevious: true,
                           modePerformancePerTestCase: true, 
                           modeOfThreshold: true,
                           failBuildIfNoResultFile: true,
                           modeThroughput: true
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
                    errorMessage = currentBuild.description ?: "No se pudo obtener el detalle del error."
                }
                
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

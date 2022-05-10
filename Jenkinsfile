pipeline {
    agent {
        docker {
            image 'hseeberger/scala-sbt:8u322_1.6.2_2.13.8'
        }
    }

    environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN')
    }

    stages {

        stage('Compile SBT Application') {
            steps {
                sh 'sbt compile'
            }
        }

        stage('Download Latest Snyk CLI') {
            steps {
                sh '''
                    snyk_cli_dl_linux="https://static.snyk.io/cli/latest/snyk-linux"
                    echo "Download URL: ${snyk_cli_dl_linux}"
                    curl -Lo ./snyk "${snyk_cli_dl_linux}"
                    chmod +x snyk
                    ls -la
                    ./snyk -v
                '''
            }
        }

        stage('Download snyk-to-html') {
            steps {
                sh '''
                    snyk_html_dl_linux="https://github.com/snyk/snyk-to-html/releases/download/v2.3.1/snyk-to-html-linux"
                    echo "Download URL: ${snyk_html_dl_linux}"
                    curl -Lo ./snyk-to-html "${snyk_html_dl_linux}"
                    chmod +x snyk-to-html
                    ls -la
                    ./snyk-to-html -h
                '''
            }
        }

        stage('Snyk Code Test using Snyk CLI') {
            steps {
                sh './snyk test --json | ./snyk-to-html -o results.html'
            }
        }

        stage('Publish Snyk Open Source Report') {
            steps {
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'results.html',
                    reportName: "Snyk Open Source Report"
                ])
            }
        }
    }
}

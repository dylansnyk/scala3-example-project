pipeline {
    agent {
        any {
            args '-u root:sudo -v $HOME/workspace/scala3-example-project:/scala3-example-project'
        }
    }

    environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN')
    }

    stages {

        stage('Install SBT') {
            steps {
                sh '''
                    apt-get update
                    apt-get install apt-transport-https curl gnupg -yqq
                    echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
                    echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt_old.list
                    curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo -H gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/scalasbt-release.gpg --import
                    chmod 644 /etc/apt/trusted.gpg.d/scalasbt-release.gpg
                    apt-get update
                    apt-get install sbt
                '''
            }
        }
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

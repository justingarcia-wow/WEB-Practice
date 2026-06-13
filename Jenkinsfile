pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Input Data') {
            steps {
                script {
                    def userInput = input(
                        message: 'Enter the data',
                        parameters: [
                            string(name:'AUTHOR', defaultValue:'Sergio'),
                            string(name:'ENVIRONMENT', defaultValue:'Development')
                        ]
                    )
                    env.AUTHOR = userInput['AUTHOR']
                    env.ENVIRONMENT = userInput['ENVIRONMENT']
                }
            }
        }

        stage('Create web directory') {
            steps {
                echo "Author: ${AUTHOR}, Env: ${ENVIRONMENT}"
                sh 'rm -rf /var/jenkins_home/web || true'
                sh 'mkdir -p /var/jenkins_home/web'
            }
        }

        stage('Drop container') {
            steps {
                sh 'docker rm -f apache1 || true'
            }
        }

        stage('Create container') {
            steps {
                sh '''
                docker run -dit \
                --name apache1 \
                -p 9000:80 \
                -v /var/jenkins_home/web:/usr/local/apache2/htdocs/ \
                httpd
                '''
            }
        }

        stage('Copy app') {
            steps {
                sh 'cp -r web/* /var/jenkins_home/web || true'
            }
        }

        stage('Test app') {
            steps {
                sh 'curl http://localhost:9000 || true'
            }
        }
    }
}

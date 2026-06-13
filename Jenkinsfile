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
                            string(name:'AUTHOR', defaultValue:'Justing'),
                            string(name:'ENVIRONMENT', defaultValue:'Development')
                        ]
                    )
                    env.AUTHOR = userInput['AUTHOR']
                    env.ENVIRONMENT = userInput['ENVIRONMENT']
                }
            }
        }

        stage('Prepare web directory') {
            steps {
                echo "Author: ${AUTHOR}, Env: ${ENVIRONMENT}"
                sh 'rm -rf /var/jenkins_home/web || true'
                sh 'mkdir -p /var/jenkins_home/web'
            }
        }

        stage('Copy app') {
            steps {
                sh '''
                echo "Contenido del workspace:"
                ls -la
                echo "Contenido de la carpeta web:"
                ls -la web

                cp -r web/* /var/jenkins_home/web
                '''
            }
        }

        stage('Create container') {
            steps {
                sh '''
                docker rm -f apache1 || true

                docker run -dit \
                --name apache1 \
                -p 9000:80 \
                -v /var/jenkins_home/web:/usr/local/apache2/htdocs/ \
                httpd
                '''
            }
        }

        stage('Verify files inside container') {
            steps {
                sh '''
                echo "Archivos dentro del contenedor:"
                docker exec apache1 ls /usr/local/apache2/htdocs/
                '''
            }
        }

        stage('Test app') {
            steps {
                sh '''
                echo "Probando aplicación..."
                curl http://host.docker.internal:9000 || true
                '''
            }
        }
    }
}

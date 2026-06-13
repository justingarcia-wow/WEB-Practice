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

        stage('Copy app to Jenkins') {
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

        stage('Create Apache container') {
            steps {
                sh '''
                echo "Eliminando contenedor anterior si existe..."
                docker rm -f apache1 || true

                echo "Creando contenedor Apache..."
                docker run -dit --name apache1 -p 9000:80 httpd
                '''
            }
        }

        stage('Copy app to container') {
            steps {
                sh '''
                echo "Copiando archivos al contenedor..."
                docker cp /var/jenkins_home/web/. apache1:/usr/local/apache2/htdocs/
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

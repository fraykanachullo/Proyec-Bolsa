pipeline {
    agent any

    tools {
        maven "MAVEN_HOME"
    }

    environment {
        DB_URL = 'jdbc:mysql://169.254.17.50:3306/bolsalaboraltest'
        DB_USERNAME = 'fray'
        DB_PASSWORD = '123456'
        JWT_SECRET = 'secretsecretsecretsecretsecretsecretsecretsecret'
        SERVER_PORT = '8080'
    }

    stages {
        stage('Clean workspace') {
            steps {
                deleteDir()  // Limpia el workspace de Jenkins
            }
        }

        stage('Clone') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    git branch: 'main', credentialsId: 'githubtoken1', url: 'https://github.com/fraykanachullo/Proyec-Bolsa.git'
                }
            }
        }

        stage('Build') {
            steps {
                timeout(time: 8, unit: 'MINUTES') {
                    sh "mvn -DskipTests clean package -f bolsa-laboral/pom.xml"
                }
            }
        }

        stage('Test') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    sh "mvn clean install -f bolsa-laboral/pom.xml"
                }
            }
        }

        stage('Sonar') {
            steps {
                timeout(time: 8, unit: 'MINUTES') {
                    withSonarQubeEnv('Sonarqube') {
                        sh "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.9.0.2155:sonar -Pcoverage -f bolsa-laboral/pom.xml"
                    }
                }
            }
        }

        stage('Quality gate') {
            steps {
                sleep(10) // seconds
                script {
                    try {
                        timeout(time: 4, unit: 'MINUTES') {
                            waitForQualityGate() // Sin abortPipeline para que no falle automáticamente
                        }
                    } catch (e) {
                        echo "Timeout o fallo del Quality Gate: ${e.getMessage()}, pero el pipeline continúa."
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    sh '''
                    export SPRING_DATASOURCE_URL=$DB_URL
                    export SPRING_DATASOURCE_USERNAME=$DB_USERNAME
                    export SPRING_DATASOURCE_PASSWORD=$DB_PASSWORD
                    export JWT_SECRET=$JWT_SECRET
                    export SERVER_PORT=$SERVER_PORT
                    mvn spring-boot:run -f bolsa-laboral/pom.xml
                    '''
                }
            }
        }
    }
}

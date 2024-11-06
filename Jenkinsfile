pipeline {
    agent any

    environment {
        // Nombre del servidor SonarQube configurado en Jenkins
        SONARQUBE_SERVER = 'SonarQube'
        // Agregar sonar-scanner al PATH
        PATH = "/opt/sonar-scanner/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                // Clonar el código fuente desde el repositorio
                checkout scm
            }
        }
        stage('SonarQube Analysis') {
            steps {
                // Configurar el entorno de SonarQube
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    // Ejecutar el análisis con SonarScanner
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=testPipeLine \
                        -Dsonar.sources=vulnerabilities \
                        -Dsonar.php.version=8.0
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                // Esperar el resultado del Quality Gate
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploy to Web Server') {
            steps {
                // Usar credenciales SSH para conectarse al servidor web
                sshagent(['webserver_ssh_credentials_id']) {
                    sh '''
                        ssh user@webserver 'cd /ruta/al/deploy && git clone https://tu.repositorio.git || (cd /ruta/al/deploy/tu_proyecto && git pull)'
                    '''
                }
            }
        }
        stage('ZAP Analysis') {
            steps {
                script {
                    // Ejecutar ZAP dentro de un contenedor Docker sin usar zap-cli
                    docker.image('owasp/zap2docker-stable').inside('--network host') {
                        sh '''
                            # Iniciar ZAP en modo demonio
                            zap.sh -daemon -host 127.0.0.1 -port 8090 -config api.disablekey=true &
                            # Esperar a que ZAP esté listo
                            timeout=120
                            while ! curl -s http://127.0.0.1:8090; do
                                sleep 5
                                timeout=$((timeout - 5))
                                if [ $timeout -le 0 ]; then
                                    echo "ZAP no se inició a tiempo"
                                    exit 1
                                fi
                            done
                            # Ejecutar el escaneo completo con zap-full-scan.py
                            zap-full-scan.py -t http://webserver/tu_proyecto -r zap_report.html -I
                            # Apagar ZAP
                            zap.sh -cmd -shutdown
                        '''
                    }
                }
                // Publicar el reporte de ZAP
                publishHTML(target: [
                    reportDir: "${env.WORKSPACE}",
                    reportFiles: 'zap_report.html',
                    reportName: 'Reporte ZAP'
                ])
            }
        }
    }
}

pipeline {
    agent any
https://github.com/herjflorez/Proyecto2-Grupo2/blob/main/Jenkinsfile
    environment {
        // Nombre del servidor SonarQube configurado en Jenkins
        SONARQUBE_SERVER = 'SonarQube-p2g2'
        SONAR_HOST_URL = 'http://10.30.212.43:9000'
        SONAR_AUTH_TOKEN = credentials('Grupo2')
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
                        -Dsonar.projectKey=Proyecto2-Grupo2 \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.token=${SONAR_AUTH_TOKEN} \
                        -Dsonar.php.version=8.0
                    '''
                }
            }
        }
        /*
        stage('Quality Gate') {
            steps {
                // Esperar el resultado del Quality Gate
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }        
        */
        stage('Deploy to Web Server') {
            steps {
                // Usar credenciales SSH para conectarse al servidor web
                sshagent(['ClaveSSH']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no root@10.30.212.43 ' cd /var/www/html && git clone https://github.com/herjflorez/Proyecto2-Grupo2.git || (cd /var/www/html/Proyecto2-Grupo2 && git pull)'
                    '''
                }
            }
        }
        stage('ZAP Analysis') {
            steps {
                sshagent(['ClaveSSH']) {
                    // Remove any existing container named 'zap_scan'
                    sh '''
                     ssh root@10.30.212.43 docker rm -f zap_scan || true
                     ssh root@10.30.212.43 docker run --user root --name zap_scan -v zap_volume:/zap/wrk/ -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t http://10.30.212.43 -P 80 -r reporte_zap.html -I -d
                     ssh root@10.30.212.43 docker cp zap_scan:/zap/wrk/reporte_zap.html /var/www/html/Proyecto2-Grupo2/reportes/reporte_zap.html
                     ssh root@10.30.212.43 docker cp /var/www/html/Proyecto2-Grupo2/reportes/reporte_zap.html jenkins:${WORKSPACE}/reporte_zap.html
                     ssh root@10.30.212.43 docker rm zap_scan
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reporte_zap.html', fingerprint: true
                }
            }
        }
    }
}

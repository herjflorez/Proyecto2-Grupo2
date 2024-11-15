pipeline {
    agent any

    environment {
        // Nombre del servidor SonarQube configurado en Jenkins
        SONARQUBE_SERVER = 'SonarQube-p2g2'
        SONAR_HOST_URL = 'http://10.30.212.62:9000'
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
                        -Dsonar.projectKey=testPipeLine \
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
                        ssh root@10.30.212.58 ' cd /var/www/html && git clone https://github.com/herjflorez/Proyecto2-Grupo2.git || (cd /var/www/html/Proyecto2-Grupo2 && git pull)'
                    '''
                }
            }
        }
        stage('ZAP Analysis') {
            steps {
                script {
                    // Remove any existing container named 'zap_scan'
                    sh '''
                    docker rm -f zap_scan || true
                    '''

                     sh '''
                    ls
                    '''

                     sh '''
                    exit
                    '''

                    sh '''
                    ls
                    '''

                    // Run OWASP ZAP container without mounting volumes and without '--rm'
                    sh '''
                    ssh root@10.30.212.58 '
                    docker run --user root --name zap_scan -v zap_volume:/zap/wrk/ -t ghcr.io/zaproxy/zaproxy:stable \
                    zap-baseline.py -t http://10.30.212.58 -P 80 \
                    -r reporte_zap.html -I -d'
                    '''

                    // Copy the report directly from the 'zap_scan' container to the Jenkins workspace
                    sh '''
                    docker cp zap_scan:/zap/wrk/reporte_zap.html ./reporte_zap.html
                    '''

                    // Remove the 'zap_scan' container
                    sh '''
                    docker rm zap_scan
                    '''
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

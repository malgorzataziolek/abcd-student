pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-pat', url: 'https://github.com/malgorzataziolek/abcd-student', branch: 'main'
                }
            }
        }
        stage('Prepare') {
            steps {
                sh 'mkdir -p results/'
            }
        }
        stage('DAST') {
            steps {
                sh '''
                    docker run --name juice-shop -d --rm \
                    -p 3000:3000 bkimminich/juice-shop
                    sleep 40
                '''
                sh '''
                    docker run --name zap \
                    -v /mnt/d/Desktop/ABC/abcd-student/zap:/zap/wrk/:rw \
                    -e JAVA_OPTS="-Xms512m -Xmx2g" \
                    -t ghcr.io/zaproxy/zaproxy:stable \
                    bash -c "zap.sh -cmd -addonupdate; \
                    zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop
                        docker rm zap
                    '''
                }
            }
        }
    } 
    post {
        always {
            echo 'Archiving artifacts...'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending artifacts to the DefectDojo..'
            defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'gosiaziolek1223@icloud.com')
        }
    }
}
            
 
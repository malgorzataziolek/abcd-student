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
        // stage('Run SCA - osv scanner'){
        //     steps{
        //         echo 'Starting osv scan...'
        //         sh 'osv-scanner scan --lockfile package-lock.json --format json --output results/osv-scanner-output.json || true'
        //     }
        // }
        stage('Trufflehog scan'){
            steps{
                echo 'Starting trufflehog scan...'
                sh 'trufflehog git file://. --branch main --only-verified --json > results/trufflehog-results.json'
            }
        }

        // stage('Run Juice Shop') {
        //     steps {
        //         echo 'Run Juice Shop'
        //         sh '''
        //             docker run --name juice-shop -d --rm \
        //             -p 3000:3000 bkimminich/juice-shop
        //             sleep 40
        //         '''
        //     }
        // }
        // stage('ZAP Passive scan') {
        //      steps {
        //         sh '''
        //             docker run --name zap -d \
        //             --add-host=host.docker.internal:host-gateway \
        //             -v /mnt/c/Users/MZ/ABC/abcd-student/.zap:/zap/wrk/:rw \
        //             -e JAVA_OPTS="-Xms512m -Xmx4g" \
        //             -t ghcr.io/zaproxy/zaproxy:stable
        //         '''
        //         sh 'sleep 20'
        //         sh '''
        //             docker exec zap zap.sh -cmd -addonupdate
        //             docker exec zap zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta
        //             docker exec zap zap.sh -cmd -autorun /zap/wrk/passive.yaml
        //         '''
        //     }
        //     post {
        //         always {
        //             sh '''
        //                 docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
        //                 docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
        //                 docker stop zap juice-shop
        //                 docker rm zap
        //             '''
        //         }
        //     }
        // }
    } 
    post {
        always {
            echo 'Archiving artifacts...'
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending artifacts to the DefectDojo..'
            // defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'gosiaziolek1223@icloud.com')
            // defectDojoPublisher(artifact: 'results/osv-scanner-output.json', productName: 'Juice Shop', scanType: 'OSV Scan', engagementName: 'gosiaziolek1223@icloud.com')
            defectDojoPublisher(artifact: 'results/trufflehog-results.json', productName: 'Juice Shop', scanType: 'Trufflehog Scan', engagementName: 'gosiaziolek1223@icloud.com')
        }
    }
}
            
 
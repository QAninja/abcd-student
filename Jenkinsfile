pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Swiezy kod z GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'devsecops', url: 'https://github.com/QAninja/abcd-student', branch: 'main'
                }
            }
        }
        stage('Przygotowanie') {
            steps {
                sh 'mkdir -p results/'
            }
        }
        // stage('Skan DAST ZAP') {
        //     steps {
        //         sh '''
        //             docker run --name juice-shop -d --rm \
        //                 -p 3000:3000 \
        //                 bkimminich/juice-shop
        //             sleep 5
        //         '''
        //         sh '''
        //             docker run --name zap \
        //             --add-host=host.docker.internal:host-gateway \
        //             -v /Users/olako/bezpiecznykod/abcd-student/.zap:/zap/wrk/:rw \
        //             -t ghcr.io/zaproxy/zaproxy:stable bash -c \
        //             "zap.sh -cmd -addonupdate && \
        //             zap.sh -cmd -addoninstall communityScripts && \
        //             zap.sh -cmd -addoninstall pscanrulesAlpha && \
        //             zap.sh -cmd -addoninstall pscanrulesBeta && \
        //             zap.sh -cmd -autorun /zap/wrk/passive.yaml" \
        //             || true
        //         '''
        //     }
        //     post {
        //         always {
        //             script{
        //             sh '''
        //                 docker cp zap:/zap/wrk/reports/zap_xml_report.xml /var/jenkins_home/workspace/ABCD/results/zap_xml_report.xml
        //             '''
        //             }
        //             defectDojoPublisher(artifact: '/var/jenkins_home/workspace/ABCD/results/zap_xml_report.xml', 
        //                 productName: 'Juice Shop', 
        //                 scanType: 'ZAP Scan', 
        //                 engagementName: 'aleksandra.k.kornecka@gmail.com')
        //             script{
        //             sh '''
        //                 docker cp zap:/zap/wrk/reports/zap_xml_report.xml /var/jenkins_home/workspace/ABCD/results/zap_xml_report.xml
        //                 docker stop juice-shop
        //                 docker stop zap
        //                 docker rm zap
        //             '''
        //             }
        //         }
        //     }
        // }
            // post {
            //     always {
            //         // ${WORKSPACE} resolves to /var/jenkins_home/workspace/ABCD
            //         sh '''
            //             docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
            //             docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
            //             docker stop zap 
            //             docker rm zap
            //         '''
            //     }
            // } 
        stage('Analiza skladu SCA z OSV') {
            steps {
                sh 'osv-scanner scan --lockfile package-lock.json --format json > /var/jenkins_home/workspace/ABCD/results/sca-osv-scanner.json'
            }
            post {
                always {
                    defectDojoPublisher(artifact: '/var/jenkins_home/workspace/ABCD/results/sca-osv-scanner.json', 
                        productName: 'Juice Shop', 
                        scanType: 'OSV Scan', 
                        engagementName: 'aleksandra.k.kornecka@gmail.com')
            }
        }
        }
        stage('Skan SAST sekretow z Trufflehog') {
            steps {
                sh 'trufflehog filesystem --directory /Users/olako/bezpiecznykod/abcd-student --only-verified --branch main --json > /var/jenkins_home/workspace/ABCD/results/trufflehog-results.json'
            }
        //     post {
        //         always {
        //             defectDojoPublisher(artifact: '/var/jenkins_home/workspace/ABCD/results/secrets-trufflehog-scan.json', 
        //                 productName: 'Juice Shop', 
        //                 scanType: 'Trufflehog Scan', 
        //                 engagementName: 'aleksandra.k.kornecka@gmail.com')
        //     }
        // }
        }
        stage('Skan SAST podatnosci z Semgrep') {
            steps {
                sh 'semgrep scan /Users/olako/bezpiecznykod/abcd-student > /var/jenkins_home/workspace/ABCD/results/vulnerabilities-semgrep-scan.json'
            }
        //     post {
        //         always {
        //             defectDojoPublisher(artifact: '/var/jenkins_home/workspace/ABCD/results/vulnerabilities-semgrep-scan.json', 
        //                 productName: 'Juice Shop', 
        //                 scanType: 'Semgrep JSON Report', 
        //                 engagementName: 'aleksandra.k.kornecka@gmail.com')
        //     }
        // }
        }
    }
}

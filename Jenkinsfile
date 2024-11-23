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
                    git credentialsId: 'devsecops', url: 'https://github.com/QAninja/abcd-student', branch: 'main'
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
                // sh 'mkdir -p results/'
                sh '''
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap --rm \
                    --add-host=host.docker.internal:host-gateway \
                    -v /Users/olako/bezpiecznykod/abcd-student/.zap/passive.yaml:/zap/wrk/passive_scan.yaml:rw \
                    -v /Users/olako/Downloads/Reports/:/zap/wrk/reports \
                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "zap.sh -cmd -addonupdate && \
                    zap.sh -cmd -addoninstall communityScripts && \
                    zap.sh -cmd -addoninstall pscanrulesAlpha && \
                    zap.sh -cmd -addoninstall pscanrulesBeta && \
                    zap.sh -cmd -autorun /zap/wrk/passive_scan.yaml"
                '''
            }
            post {
                always {
                    // ${WORKSPACE} resolves to /var/jenkins_home/workspace/ABCD
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop
                        docker rm zap
                    '''
                }
            }
            // post {
            //     always {
            //         defectDojoPublisher(artifact: 'results/zap_xml_report.xml', 
            //             productName: 'Juice Shop', 
            //             scanType: 'ZAP Scan', 
            //             engagementName: 'aleksandra.k.kornecka@gmail.com')
            //     }
            // }
        }
        stage('SCA') {
            steps {
                sh 'osv-scanner scan --lockfile package-lock.json --format json --output results/sca-osv-scanner.json'
            }
        //     post {
        //         always {
        //             defectDojoPublisher(artifact: 'results/sca-osv-scanner.json', 
        //                 productName: 'Juice Shop', 
        //                 scanType: 'OSV Scan', 
        //                 engagementName: 'aleksandra.k.kornecka@gmail.com')
        //     }
        // }
        }
        stage('Secrets') {
            steps {
                sh 'trufflehog docker --image=bkimminich/juice-shop --format json --output results/secrets-trufflehog-scan.json'
            }
        //     post {
        //         always {
        //             defectDojoPublisher(artifact: 'results/secrets-trufflehog-scan.json', 
        //                 productName: 'Juice Shop', 
        //                 scanType: 'Trufflehog Scan', 
        //                 engagementName: 'aleksandra.k.kornecka@gmail.com')
        //     }
        // }
        }
        stage('Vulnerabilities') {
            steps {
                sh 'semgrep scan /Users/olako/bezpiecznykod/abcd-student --format json --output results/vulnerabilities-semgrep-scan.json'
            }
        //     post {
        //         always {
        //             defectDojoPublisher(artifact: 'results/vulnerabilities-semgrep-scan.json', 
        //                 productName: 'Juice Shop', 
        //                 scanType: 'Semgrep JSON Report', 
        //                 engagementName: 'aleksandra.k.kornecka@gmail.com')
        //     }
        // }
        }
    }
}

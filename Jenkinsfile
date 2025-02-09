node {
    checkout scm
    docker.image('python:2-alpine').inside {    
        stage('Build') {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }
    docker.image('qnib/pytest').inside { 
        stage('Test') {
            try {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            finally {
                junit 'test-reports/results.xml'
            }
        }
    }
    docker.image('batonogov/pyinstaller-linux').inside("--entrypoint=''") {
        stage('Deploy') {
            try {
                sh 'pyinstaller --onefile sources/add2vals.py'
                // delivers the website into EC2 instance via SSH
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'ec2-st-server-1', 
                        transfers: [
                            sshTransfer
                            (
                                cleanRemote: false, 
                                excludes: '', 
                                execCommand: 'pwd && cd /home/ubuntu/.deployments/python-app && pwd && chmod +x add2vals && ./add2vals 1 1', 
                                execTimeout: 120000, 
                                flatten: false, 
                                makeEmptyDirs: false, 
                                noDefaultExcludes: false, 
                                patternSeparator: '[, ]+', 
                                remoteDirectory: '/home/ubuntu/.deployments/python-app', 
                                remoteDirectorySDF: false, 
                                removePrefix: 'dist', 
                                sourceFiles: 'dist/add2vals'
                            )
                        ], 
                        usePromotionTimestamp: false, 
                        useWorkspaceInPromotion: false, 
                        verbose: true
                    )
                ])
            }
            finally {
                archiveArtifacts 'dist/add2vals'
            }
        }
    }
}
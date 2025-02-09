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
    stage('Manual Approval') {
        try {
            input message: 'Continue to Deploy?'
        } catch (exception) {
            echo 'Manual Approval : Aborted'
            throw exception
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
                                execCommand: 'cd /home/ubuntu/.deployments/python-app && chmod +x add2vals && ./add2vals 1 1', 
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
                try {
                    timeout(time: 60, unit: 'SECONDS') {
                        input message: 'Finished using the app? (Click "Proceed" to continue or wait 1 minute to finish & keep compiled file, abort will finish & delete the file)'
                    }
                } catch (err) { 
                    echo 'Deploy : Deployment aborted'
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: 'ec2-st-server-1', 
                            transfers: [
                                sshTransfer
                                (
                                    execCommand: 'rm -rf /home/ubuntu/.deployments/python-app'
                                )
                            ], 
                            usePromotionTimestamp: false, 
                            useWorkspaceInPromotion: false, 
                            verbose: true
                        )
                    ])
                    echo 'Deploy : Compiled file deleted. Pipeline finished'
                }
                echo 'Deploy : File persisted. Pipeline finished'
                echo 'Pipeline finished.'
            }
            finally {
                archiveArtifacts 'dist/add2vals'
            }
        }
    }
}
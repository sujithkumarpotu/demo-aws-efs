pipeline {
    agent {
        label 'master'
    }
 
    parameters {
            string(name: 'EFS_PATH', defaultValue: '', description: 'EFS folder path (ex. myfolder) or leave empty for root path', trim: true)
    }
 
    environment {
        CODEFLEX_AWS_ACCOUNT_ID = '77777777777'
        CODEFLEX_REGION = 'eu-west-1'
        SECRET_ID = 'codeflex-rds'
    }
 
    options {
        timestamps()
        ansiColor('xterm')
        skipDefaultCheckout true
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '25', artifactNumToKeepStr: '25'))
    }
 
    stages {
        
        stage('Mount EFS') {
            steps {
                script {
                    withAWS(role: 'codeflex-jenkins', roleAccount: env.CODEFLEX_AWS_ACCOUNT_ID, region: env.CODEFLEX_REGION) {
						sh ('sudo apt-get -y install nfs-utils')
                        sh('mkdir -p ~/efs-mount-point')
                        sh('sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport 172.28.81.41:/ ~/efs-mount-point')
                        sh('sudo chown -R jenkins:jenkins ~/efs-mount-point')
                    }
                }
            }
        }
 
        stage('Check Mount') {
            steps {
                script {
                    withAWS(role: 'codeflex-jenkins', roleAccount: env.CODEFLEX_AWS_ACCOUNT_ID, region: env.CODEFLEX_REGION) {
                        echo '**********************************************'
                        sh('df -T')
                        echo '**********************************************'
                        sh('ls -l ~/efs-mount-point')
                        echo '**********************************************'
                    }
                }
            }
        }
 
        stage('EFS Path Validation') {
            steps {
                script {
                    if ("${params.EFS_PATH}".length() > 0) {
                        efs_path = '~/efs-mount-point/' + params.EFS_PATH
 
                        stdout = sh( script: '''#!/bin/bash
                                set -ex
                                if [ -d ''' + efs_path + ''' ]; then
                                    echo "true"
                                else
                                    echo "false"
                                fi
                                ''', returnStdout: true)
 
                        echo stdout
 
                        if (stdout.contains('false')) {
                            error('No such directory on EFS: ' + efs_path)
                        }
                    }
                }
            }
        }
 
        stage('Copy From S3 to EFS') {
            steps {
                script {
                    withAWS(role: 'codeflex-jenkins', roleAccount: env.CODEFLEX_AWS_ACCOUNT_ID, region: env.CODEFLEX_REGION) {
                        echo '**********************************************'
                        sh("ls -l ${final_efs_path}")
                        echo '**********************************************'
                    }
                }
            }
        }
    }
 
    post {
        always {
            deleteDir()
            script {
                sh('sudo umount ~/efs-mount-point')
                sh('sudo rm -rf ~/efs-mount-point')
            }
        }
    }
}

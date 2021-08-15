pipeline {
    agent {
        label 'master'
    }
 
    parameters {
            string(name: 'EFS_PATH', defaultValue: '', description: 'EFS folder path (ex. myfolder) or leave empty for root path', trim: true)
    }
 
    environment {
        AWS_ACCESS_KEY_ID = 'AKIAXKDDYNSPFJVR3HIR'
	AWS_SECRET_ACCESS_KEY = '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'
        AWS_REGION = 'eu-west-1'
        SECRET_ID = ''
	GIT_URL = "https://github.com/sujithkumarpotu/demo-aws-efs.git"
	GIT_BRANCH = "main"
    }
 
    options {
        timestamps()
        skipDefaultCheckout true
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '25', artifactNumToKeepStr: '25'))
    }
 
    stages {
        
        stage('Mount EFS') {
            steps {
                script {
		   sh'''
		   sudo apt update
		   aws configure set region AWS_REGION
		   sudo apt-get -y install nfs-kernel-server nfs-common awscli
                   mkdir -p ~/efs-mount-point
                   sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport 172.31.20.15:/ ~/efs-mount-point
                   sudo chown -R jenkins:jenkins ~/efs-mount-point
		   '''
                }
            }
        }
 
        stage('Check Mount') {
            steps {
                script {
                    echo '**********************************************'
                    sh('df -T')
                    echo '**********************************************'
                    sh('ls -l ~/efs-mount-point')
		    final_efs_path = '~/efs-mount-point'
                    echo '**********************************************'
                }
            }
        }
 
        stage('EFS Path Validation') {
            steps {
                script {
                        efs_path = '~/efs-mount-point/' 
 
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

	stage ('Checkout') {
            steps {
                script {
                echo "Checking out release ${GIT_BRANCH}"
                checkout (
                    [
                    $class: 'GitSCM',
                    branches: [
                        [name: GIT_BRANCH]
                    ],
                    doGenerateSubmoduleConfigurations: false,
                    submoduleCfg: [],
                    userRemoteConfigs: [
                        [
                        // credentialsId: GIT_CREDENTIALS,
                        url: GIT_URL
                        ]
                    ]
                    ]
                )
                }
            }
        }
 
        stage('Copy From from airflow/dags to EFS') {
            steps {
                script {
                     echo '**********************************************'
                     echo 'Current directory files'
		     sh("ls -lrtA ")
		     echo 'Mount locatin files'
                     sh("ls -l ${final_efs_path}")
		     sh("cp -pR airflow/dags/*.py ${final_efs_path}/dags")
		     sh("ls -lrA ${final_efs_path}/dags/")
		     sh('sudo chown -R root:root ~/efs-mount-point')
                     echo '**********************************************'
                }
            }
        }
    }
 
    post {
        always {
            deleteDir()
            script {
		echo "Delete mount and efs-mount-point directory"
                // sh('sudo umount ~/efs-mount-point')
                // sh('sudo rm -rf ~/efs-mount-point')
            }
        }
    }
}

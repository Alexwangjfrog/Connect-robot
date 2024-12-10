pipeline {
    agent any
    tools {
        jfrog 'jfrog-cli-demojfrogchina'
    }
    triggers {
        // 使用 webhook 触发
        pollSCM('') // 留空表示仅响应 Webhook
    }
    stages {
        stage('CLI Testing') {
            steps {
                // Show the installed version of JFrog CLI.
                jf '-v'

                // Show the configured JFrog Platform instances.
                jf 'c show'

                // Ping Artifactory.
                jf 'rt ping'
            }
        }

        stage('Clone') {
            steps {
                git url: 'https://gitee.com/mumu79/simulated-robot.git', branch: 'master'
            }
        }

        stage('Build') {
            steps {
                // Upload artifact to Artifactory
                jf 'rt u simulated_vacuum_robot.py slash-generic-test-local/vacuum_robot/v' + env.BUILD_NUMBER + '/ --build-name ' + env.JOB_NAME + ' --build-number ' + env.BUILD_NUMBER
            }
        }

        stage('Publish build info') {
            steps {
                jf 'rt build-publish ' + env.JOB_NAME  + ' ' + env.BUILD_NUMBER
            }
        }

        // stage('Xray scan') {
        //     steps {
        //         jf 'bs ' + env.JOB_NAME  + ' ' + env.BUILD_NUMBER + ' --fail=false'
        //     }
        // }

        stage('Promotion') {
            steps {
                echo "Promote to release repo"
                echo env.JOB_NAME
                echo env.BUILD_NUMBER

                jf 'rt bpr ' + env.JOB_NAME  + ' ' + env.BUILD_NUMBER  + ' slash-generic-release-local --status=RELEASE --comment=Release_App_robot --source-repo=slash-generic-test-local --copy=true'

            }
        }

        stage('Set to release') {
            steps {
                sleep time: 2, unit: 'SECONDS' // 暂停 2 秒，等待晋级完成

                // Install and publish project
                jf 'rt sp slash-generic-release-local/vacuum_robot/v' + env.BUILD_NUMBER + '/simulated_vacuum_robot.py v' + env.BUILD_NUMBER + '=version'
            }
        }
        
    }
}

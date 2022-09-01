node("built-in") {
    stage("Checkout") {
        checkout scm

        sh 'ansible-galaxy install -f -r requirements.yml'
    }

    try {
        stage("Create") {
            withCredentials([usernamePassword(
                    credentialsId: 'jenkins_infra_account',
                    usernameVariable: 'VSPHERE_USER',
                    passwordVariable: 'VSPHERE_PASSWORD'
            )]) {
                sh "cd ./roles/setup && molecule reset -s install && molecule create -s install"
            }
        }

        stage("Install") {
            withCredentials([usernamePassword(
                    credentialsId: 'jenkins_infra_account',
                    usernameVariable: 'VSPHERE_USER',
                    passwordVariable: 'VSPHERE_PASSWORD'
            )]) {


                install_stages = [:]

                install_stages['install-ubuntu'] = {
                    sh "cd ./roles/setup && molecule test -s install-ubuntu --parallel" //&& molecule verify -s install-ubuntu"
                }

                install_stages['install-centos'] = {
                    sh "cd ./roles/setup && molecule test -s install-centos --parallel" // && molecule verify -s install-centos"
                }

                parallel install_stages
            }
        }

        stage("Deploy") {
            withCredentials([usernamePassword(
                    credentialsId: 'jenkins_infra_account',
                    usernameVariable: 'VSPHERE_USER',
                    passwordVariable: 'VSPHERE_PASSWORD'
            )]) {
                sh "cd ./roles/use && molecule reset -s deploy"
                sh "cd ./roles/use && molecule converge -s deploy && molecule verify -s deploy"
            }
        }

        stage("Undeploy") {
            withCredentials([usernamePassword(
                    credentialsId: 'jenkins_infra_account',
                    usernameVariable: 'VSPHERE_USER',
                    passwordVariable: 'VSPHERE_PASSWORD'
            )]) {
                sh "cd ./roles/use && molecule reset -s undeploy"
                sh "cd ./roles/use && molecule converge -s undeploy && molecule verify -s undeploy"
            }
        }

        stage("Uninstall") {
            withCredentials([usernamePassword(
                    credentialsId: 'jenkins_infra_account',
                    usernameVariable: 'VSPHERE_USER',
                    passwordVariable: 'VSPHERE_PASSWORD'
            )]) {
                sh "cd ./roles/setup && molecule reset -s uninstall"
                sh "cd ./roles/setup && molecule converge -s uninstall && molecule verify -s uninstall"
            }
        }
    } finally {
        stage("Destroy") {
            withCredentials([usernamePassword(
                    credentialsId: 'jenkins_infra_account',
                    usernameVariable: 'VSPHERE_USER',
                    passwordVariable: 'VSPHERE_PASSWORD'
            )]) {
                sh "cd ./roles/setup && molecule destroy -s uninstall"
                sh "cd ./roles/setup && molecule reset -s install-ubuntu"
                sh "cd ./roles/setup && molecule reset -s install-centos"
            }
        }
    }
}
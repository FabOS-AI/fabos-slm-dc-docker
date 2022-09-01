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
                sh "cd ./roles/setup && molecule reset -s install"

                parallel install-ubuntu: {
                    sh "cd ./roles/setup && molecule converge -s install-ubuntu && molecule verify -s install-ubuntu"
                }, install-centos: {
                    sh "cd ./roles/setup && molecule converge -s install-centos && molecule verify -s install-centos"
                }

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
            }
        }
    }
}
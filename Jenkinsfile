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
                    sh 'docker run -v \$(pwd):/tmp/roles --env VSPHERE_PASSWORD=$VSPHERE_PASSWORD --env VSPHERE_HOSTNAME=$VSPHERE_HOSTNAME --env VSPHERE_USER=$VSPHERE_USER -w /tmp/roles/roles/setup --rm fabos4ai/molecule:4.0.1  bash -c "ansible-galaxy install -f -r /tmp/roles/requirements.yml && molecule converge -s install-ubuntu"'

//                    sh "cd ./roles/setup && molecule test -s install-ubuntu --parallel" //&& molecule verify -s install-ubuntu"
                }

                install_stages['install-centos'] = {
                    sh 'docker run -v \$(pwd):/tmp/roles --env VSPHERE_PASSWORD=$VSPHERE_PASSWORD --env VSPHERE_HOSTNAME=$VSPHERE_HOSTNAME --env VSPHERE_USER=$VSPHERE_USER -w /tmp/roles/roles/setup --rm fabos4ai/molecule:4.0.1  bash -c "ansible-galaxy install -f -r /tmp/roles/requirements.yml && molecule converge -s install-centos"'
//                    sh "cd ./roles/setup && molecule test -s install-centos --parallel" // && molecule verify -s install-centos"
//                    sh "pytest roles/setup/molecule/install-centos/molecule.yml"
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
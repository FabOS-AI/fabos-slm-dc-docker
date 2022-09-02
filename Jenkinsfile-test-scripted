def scenarios = [
    "ubuntu2204": [
        ["setup", "install"],
        ["use", "deploy"],
        ["use", "undeploy"],
        ["setup", "uninstall"]
    ],
    "ubuntu2004": [
        ["setup", "install"],
        ["use", "deploy"],
        ["use", "undeploy"],
        ["setup", "uninstall"]
    ],
     "ubuntu1804": [
         ["setup", "install"],
         ["use", "deploy"],
         ["use", "undeploy"],
         ["setup", "uninstall"]
     ],
    "centos8": [
       ["setup", "install"],
       ["use", "deploy"],
       ["use", "undeploy"],
       ["setup", "uninstall"]
    ],
     "centos7": [
        ["setup", "install"],
        ["use", "deploy"],
        ["use", "undeploy"],
        ["setup", "uninstall"]
     ]
]

parallel_stages = [:]

for (kv in mapToList(scenarios)) {
    def plattform = kv[0]
    def testList = kv[1]

    parallel_stages[plattform] = {
        for(int i = 0; i < testList.size(); i++) {
            def role = testList[i][0]
            def scenario = testList[i][1]

            stage("${plattform} - ${scenario}") {
                docker.image('fabos4ai/molecule:4.0.1').inside('-u root') {
                    withCredentials([usernamePassword(
                            credentialsId: 'jenkins_infra_account',
                            usernameVariable: 'VSPHERE_USER',
                            passwordVariable: 'VSPHERE_PASSWORD'
                    )]) {
                        sh "ansible-galaxy install -f -r requirements.yml"
                        sh "cd ./roles/${role} && molecule test -s ${scenario} -p ${plattform} --destroy never"
                    }
                }
            }
        }
    }
}

node {
    checkout scm

    stage("Create") {
        withCredentials([usernamePassword(
                credentialsId: 'jenkins_infra_account',
                usernameVariable: 'VSPHERE_USER',
                passwordVariable: 'VSPHERE_PASSWORD'
        )]) {
            sh "cd ./roles/setup && molecule reset -s install && molecule create -s install"
        }
    }

    try {
        parallel(parallel_stages)
    } finally {
        stage("Destroy") {
            withCredentials([usernamePassword(
                    credentialsId: 'jenkins_infra_account',
                    usernameVariable: 'VSPHERE_USER',
                    passwordVariable: 'VSPHERE_PASSWORD'
            )]) {
                sh "cd ./roles/setup && molecule destroy -s install"
            }
        }
    }
}

@NonCPS
List<List<?>> mapToList(Map map) {
  return map.collect { it ->
    [it.key, it.value]
  }
}
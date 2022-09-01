def scenarios = [
    "master" : [
        "ubuntu": [
            ["setup", "install-ubuntu"],
            ["use", "deploy-ubuntu"],
            ["use", "undeploy-ubuntu"],
            ["setup", "uninstall-ubuntu"],
        ]
    ],
    "Agent-1" : [
        "centos" : [
            ["setup", "install-centos"],
            ["use", "deploy-centos"],
            ["use", "undeploy-centos"],
            ["setup", "uninstall-centos"],
        ]
    ],
]

def nodes = [:]

for (kv in mapToList(scenarios)) {
    def nodeName = kv[0]
    def distList = kv[1]

    nodes[nodeName] = {
        node("${nodeName}") {

            /**
            *
            *   CHECKOUT
            *
            **/

            stage("Checkout") {
                checkout scm

                sh 'ansible-galaxy install -f -r requirements.yml'
            }

            for (xy in mapToList(distList)) {
                def distName = xy[0]
                def scenarioList = xy[1]

                try {
                    /**
                    *
                    *   CREATE
                    *
                    **/

                    def createRole = scenarioList.first()[0]
                    def createScenario = scenarioList.first()[1]

                    stage("Create - ${distName}") {
                        withEnv([
                        'HYPERVISOR_ANSIBLE_USER=',
                        'HYPERVISOR_ANSIBLE_HOST=localhost',
                        'HYPERVISOR_ANSIBLE_CONNECTION=local',
                        'HYPERVISOR_ANSIBLE_PASSWORD=',
                        'ANSIBLE_SERIAL=5',
                        ]) {
                            retry(2) {
                                sh "cd ./roles/${createRole} && molecule reset -s ${createScenario} && molecule create -s ${createScenario}"
                            }
                        }
                    }

                    /**
                    *
                    *   INSTALL, DEPLOY, UNDEPLOY, UNINSTALL
                    *
                    **/

                    for(int i = 0; i < scenarioList.size(); i++) {
                        def role = scenarioList[i][0]
                        def scenario = scenarioList[i][1]

                        stage("${scenario}") {
                            withEnv([
                            'HYPERVISOR_ANSIBLE_USER=',
                            'HYPERVISOR_ANSIBLE_HOST=localhost',
                            'HYPERVISOR_ANSIBLE_CONNECTION=local',
                            'HYPERVISOR_ANSIBLE_PASSWORD=',
                            'ANSIBLE_SERIAL=5',
                            ]) {
                                sh "cd ./roles/${role} && molecule converge -s ${scenario} && molecule verify -s ${scenario}"
                            }
                        }
                    }
                } catch(all) {
                    echo "${distName} failed."
                } finally {

                    /**
                    *
                    *   DESTROY
                    *
                    **/

                    def destroyRole = scenarioList.last()[0]
                    def destroyScenario = scenarioList.last()[1]

                    stage("Destroy - ${distName}") {
                        withEnv([
                        'HYPERVISOR_ANSIBLE_USER=',
                        'HYPERVISOR_ANSIBLE_HOST=localhost',
                        'HYPERVISOR_ANSIBLE_CONNECTION=local',
                        'HYPERVISOR_ANSIBLE_PASSWORD=',
                        'ANSIBLE_SERIAL=5',
                        ]) {
                            sh "cd ./roles/${destroyRole} && molecule destroy -s ${destroyScenario}"
                        }
                    }
                }
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

parallel nodes
#!groovy

// pod utilisé pour la compilation du projet
podTemplate(label: 'chart-run-pod', containers: [

        // le slave jenkins
        containerTemplate(name: 'jnlp', image: 'jenkinsci/jnlp-slave:alpine'),

        containerTemplate(name: 'helm', image: 'elkouhen/k8s-helm:2.9.1d', ttyEnabled: true, command: 'cat')],

        // montage nécessaire pour que le conteneur docker fonction (Docker In Docker)
        volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) {

    node('chart-run-pod') {

        def charts = ['books-api', 'books-gui', 'h2', 'grav', 'jenkins-impl', 'nexus-impl', 'sonarqube-impl', 'elasticstack']

        def envs = ['dev', 'prod']

        properties([
                parameters([
                        string(defaultValue: 'latest', description: 'Version à déployer', name: 'image'),
                        choice(choices: charts, description: 'Chart à deployer', name: 'chart'),
                        choice(choices: envs, description: 'Environnement de déploiement', name: 'env'),
                ])
        ])

        stage('CHECKOUT') {
            checkout scm;
        }

        container('helm') {

            stage('DEPLOY') {

                withCredentials([string(credentialsId: 'pgp_helm_pwd', variable: 'pgp_helm_pwd')]) {

                    configFileProvider([configFile(fileId: 'pgp-helm', targetLocation: "secret.asc")

                    ]) {

                        String command = "./deploy.sh -p ${pgp_helm_pwd} -c ${params.chart} "

                        if (params.env != '') {
                            command += "-e ${params.env} "
                        }

                        if (params.image != '') {
                            command += "-i ${params.image} "
                        }

                        sh "${command}"
                    }
                }
            }
        }
    }
}

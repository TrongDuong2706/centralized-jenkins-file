def call(Map config = [:]) {
    ['appRepo', 'image', 'deployHost', 'sshCredentialsId'].each {
        requireValue(config, it)
    }
    if (config.deployHost.toString() == 'CHANGE_ME') {
        error 'Set deployHost in todolist/todolist-cd/config.yaml before running CD.'
    }

    String imageRef = "${config.image}:${config.imageTag ?: 'latest'}"

    stage('Checkout Deployment Compose File') {
        dir('application') {
            def remote = [url: config.appRepo.toString()]
            if (config.appGitCredentialsId) {
                remote.credentialsId = config.appGitCredentialsId.toString()
            }
            checkout([
                $class: 'GitSCM',
                branches: [[name: "*/${config.appBranch ?: 'main'}"]],
                extensions: [],
                userRemoteConfigs: [remote]
            ])
        }
    }

    stage('Deploy Frontend') {
        dir('application') {
            sshDockerComposeDeploy(
                host: config.deployHost.toString(),
                port: (config.deployPort ?: 22).toString(),
                sshCredentialsId: config.sshCredentialsId.toString(),
                image: imageRef,
                composeFile: (config.composeFile ?: 'docker-compose.yml').toString(),
                deployDirectory: (config.deployDirectory ?: '/opt/todolist').toString(),
                service: (config.service ?: 'frontend').toString(),
                registry: (config.registry ?: 'docker.io').toString(),
                registryCredentialsId: config.registryCredentialsId?.toString()
            )
        }
    }

    currentBuild.description = "Deployed ${imageRef} to ${config.deployHost}"
}

private void requireValue(Map config, String key) {
    if (!config[key]) {
        error "Vue CD configuration requires '${key}'."
    }
}

// `load` returns the result of this script. Return the script instance so the
// caller can invoke its `call(config)` method.
return this

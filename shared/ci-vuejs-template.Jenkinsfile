def call(Map config = [:]) {
    requireValue(config, 'appRepo')
    requireValue(config, 'image')
    requireValue(config, 'registryCredentialsId')

    String imageRef

    stage('Checkout Application') {
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

    stage('Build Frontend Image') {
        dir('application') {
            imageRef = dockerBuild(
                image: config.image.toString(),
                tag: (config.imageTag ?: 'latest').toString(),
                context: (config.buildContext ?: 'frontend').toString(),
                dockerfile: config.dockerfile?.toString()
            )
        }
    }

    stage('Push Frontend Image') {
        dockerPush(
            image: imageRef,
            registry: (config.registry ?: 'docker.io').toString(),
            credentialsId: config.registryCredentialsId.toString()
        )
    }

    currentBuild.description = "Pushed ${imageRef}"
    return imageRef
}

private void requireValue(Map config, String key) {
    if (!config[key]) {
        error "Vue CI configuration requires '${key}'."
    }
}

// `load` returns the result of this script. Return the script instance so the
// caller can invoke its `call(config)` method.
return this

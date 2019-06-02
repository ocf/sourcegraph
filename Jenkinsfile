    pipeline {
        // TODO: Make this cleaner: https://issues.jenkins-ci.org/browse/JENKINS-42643
        triggers {
            upstream(
                upstreamProjects: (env.BRANCH_NAME == 'master' ? pipelineParams.upstreamProjects?.join(',') : ''),
                threshold: hudson.model.Result.SUCCESS,
            )
        }

        agent {
            label 'slave'
        }

        options {
            timeout(time: 1, unit: 'HOURS')
            timestamps()
        }

        stages {
            stage('deploy-to-prod') {
                agent {
                    label 'deploy'
                }
                steps {
                    script {
                        if (fileExists('kubernetes')) {
                            kubernetesDeployApp('sourcegraph', '5')
                        }
                    }
                }
            }
        }

        post {
            failure {
                emailNotification()
            }
            always {
                node(label: 'slave') {
                    ircNotification()
                }
            }
        }
    }

// vim: ft=groovy

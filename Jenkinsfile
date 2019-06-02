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
            stage('check-gh-trust') {
                steps {
                    checkGitHubAccess()
                }
            }


            stage('deploy-to-prod') {
                agent {
                    label 'deploy'
                }
                steps {
                    script {
                        // TODO: Make these deploy and roll back together!
                        // TODO: Also try to parallelize these if possible?
                        pipelineParams.deployTargets.each {
                            marathonDeployApp(it, version)
                        }

                        if (fileExists('kubernetes')) {
                            kubernetesDeployApp(repo_name, version)
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

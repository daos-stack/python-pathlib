pipeline {
    agent none

    stages {
        stage('Lint') {
            stages {
                stage('RPM Lint') {
                    agent {
                        dockerfile {
                            filename 'Dockerfile.centos.7'
                            label 'docker_runner'
                            additionalBuildArgs  '--build-arg UID=$(id -u)'
                            args  '--group-add mock --cap-add=SYS_ADMIN --privileged=true'
                        }
                    }
                    steps {
                        sh 'echo "linting on EL7 not supported for this RPM"'
                    }
                }
            }
        }
        stage('Build') {
            parallel {
                stage('Build on SLES 12.3') {
                    agent {
                        dockerfile {
                            filename 'Dockerfile.sles.12.3'
                            label 'docker_runner'
                            additionalBuildArgs  '--build-arg UID=$(id -u) ' +
                                                 "--build-arg CACHEBUST=${currentBuild.startTimeInMillis}"
                        }
                    }
                    steps {
                        sh '''rm -rf artifacts/sles12.3/
                              mkdir -p artifacts/sles12.3/
                              rm -rf _topdir/SRPMS
                              if make srpm; then
                                  rm -rf _topdir/RPMS
                                  if make rpms; then
                                      ln _topdir/{RPMS/*,SRPMS}/*  artifacts/sles12.3/
                                      createrepo artifacts/sles12.3/
                                  else
                                      exit \${PIPESTATUS[0]}
                                  fi
                              else
                                  exit \${PIPESTATUS[0]}
                              fi'''
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'artifacts/sles12.3/**'
                        }
                    }
                }
                stage('Build on Ubuntu 18.04') {
                    agent {
                        label 'docker_runner'
                    }
                    steps {
                        echo "Building on Ubuntu is not implemented for the moment"
                    }
                }
            }
        }
    }
}

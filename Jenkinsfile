properties([
  [
    $class: 'ThrottleJobProperty',
    categories: ['category'],
    limitOneJobWithMatchingParams: false,
    maxConcurrentPerNode: 4,
    maxConcurrentTotal: 0,
    paramsToUseForLimit: '',
    throttleEnabled: true,
    throttleOption: 'category'
  ],
])

pipeline {
    agent {
/*
        dockerfile {
            label 'slave'
        }
*/
        label 'slave'
    }

    stages {
        stage('Prepare') {
            steps {
                sh 'docker build -t stdcompat .'
            }
        }
        stage('Build') {
            steps {
                sh 'docker run -v $PWD:/workspace stdcompat sh -c \'cd /workspace && eval `opam config env` && autoreconf && mkdir build && cd build && ../configure && make\''
            }
        }
        stage('Test') {
            steps {
                script {
                    def switches = sh (
                        script: 'docker run stdcompat opam switch -i -s',
                        returnStdout: true
                    ).split('\n')
                    def branches = [:]
                    for (i in switches) {
                        def switch_name = i
                        branches[switch_name] = {
                            node {
                                sh "docker run -v $PWD:/workspace stdcompat sh -c 'cd /workspace && opam config exec --switch $switch_name -- sh -c '\''mkdir build/$switch_name && cd build/$switch_name && ../../configure && make && make tests'\'"
                            }
                        }
                    }
                    throttle(['category']) {
                        parallel branches
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'cd build && make dist'
                archiveArtifacts artifacts: 'build/*.tar.gz', fingerprint: true
            }
        }
    }
}

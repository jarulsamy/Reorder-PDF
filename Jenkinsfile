pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        // Keep the 2 most recent builds
        buildDiscarder(logRotator(numToKeepStr: '2'))
        timestamps()
    }

    environment {
      PATH="/var/lib/jenkins/miniconda3/bin:$PATH"
    }

    stages {
        stage ("Code pull"){
            steps{
                checkout scm
            }
        }

        stage('Build environment') {
            steps {
                echo "Building virtualenv"
                sh  ''' conda create --yes -n ${BUILD_TAG} python
                        source /var/lib/jenkins/miniconda3/etc/profile.d/conda.sh
                        conda activate ${BUILD_TAG}
                        pip install pylint pyinstaller
                    '''
            }
        }

        stage('Static code metrics') {
            steps {
                echo "Style check"
                sh  ''' source /var/lib/jenkins/miniconda3/etc/profile.d/conda.sh
                        conda activate ${BUILD_TAG}
                        pylint **/*.py || true
                    '''
            }
        }

        stage('Build module') {
            when {
                expression {
                    currentBuild.result == null || currentBuild.result == 'SUCCESS'
                }
            }
            steps {
                sh  ''' source /var/lib/jenkins/miniconda3/etc/profile.d/conda.sh
                        conda activate ${BUILD_TAG}
                        python setup.py bdist_wheel
                    '''
            }
            post {
                always {
                    // Archive unit tests for the future
                    archiveArtifacts (allowEmptyArchive: true,
                                     artifacts: 'dist/*whl',
                                     fingerprint: true)
                }
            }
        }

        // stage('Build standalone executable'){
        //     when {
        //         expression {
        //             currentBuild.result == null || currentBuild == 'SUCCESS'
        //         }
        //     }
        //     steps {
        //         sh  ''' source /var/lib/jenkins/miniconda3/etc/profile.d/conda.sh
        //                 conda activate ${BUILD_TAG}
        //                 python build_standalone_exe.py
        //             '''
        //     }
        //     post {
        //         always {
        //             // Archive unit tests for the future
        //             archiveArtifacts (allowEmptyArchive: true,
        //                              artifacts: 'dist/ReorderPDF',
        //                              fingerprint: true)
        //         }
        //     }
        // }
    }

    post {
        always {
            sh 'conda remove --yes -n ${BUILD_TAG} --all'
        }
        failure {
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                         <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }
    }
}

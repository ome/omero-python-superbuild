pipeline {
    agent {
        label 'testintegration'
    }

    environment {
        // Default credentials for testing on devspace
        MAVEN_SNAPSHOTS_REPO_URL = 'http://nexus:8081/nexus/repository/maven-internal/'
        MAVEN_USER = 'admin'
        MAVEN_PASSWORD = 'admin123'

        // Disable Gradle daemon
        GRADLE_OPTS = '-Dorg.gradle.daemon=false'
    }

    stages {
        stage('Versions') {
            steps {

                copyArtifacts(projectName: 'OMERO-build-build', flatten: true)

                // build is in .gitignore so we can use it as a temp dir
                sh """
                    mkdir ${env.WORKSPACE}/build
                    cd ${env.WORKSPACE}/build && curl -sfL https://github.com/ome/build-infra/archive/master.tar.gz | tar -zxf -
                    export PATH=$PATH:${env.WORKSPACE}/build/build-infra-master/
                    cd ..
                    # Workaround for "unflattened" file, possibly due to matrix
                    find . -name version.properties -exec cp {} . \\;

                    test -e version.properties
                    # First download of omero-blitz happens here
                    export VERSION_PROPERTIES=${env.WORKSPACE}/version.properties
                    export ZIP_FILE=${env.WORKSPACE}/omero-blitz-VERSION-python.zip
                    foreach-get-version-as-property >> version.properties
                """
                archiveArtifacts artifacts: 'version.properties'
            }
        }
        stage('Build') {
            steps {
                sh """
                    export VERSION_PROPERTIES=${env.WORKSPACE}/version.properties
                    # VERSION will be in omero-py/setup.py
                    export ZIP_FILE=${env.WORKSPACE}/omero-blitz-VERSION-python.zip
                    git submodule update --init --recursive
                    git submodule foreach python3.11 -m build
                """
                archiveArtifacts artifacts: '*/dist/*tar.gz'
            }
        }
        stage('Deploy') {
            steps {
                sh 'echo see https://help.sonatype.com/repomanager3/formats/pypi-repositories'
                sh 'echo was gradle --init-script init-ci.gradle publish'
            }
        }
    }

    post {
        always {
            // Cleanup workspace
            deleteDir()
        }
    }

}

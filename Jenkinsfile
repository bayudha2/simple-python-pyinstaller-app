node {
    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }
    stage('Test') {
        docker.image('qnib/pytest').inside {
            try {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            } catch (Exception e) {
                echo 'Error: ' + e.toString()
            } finally {
                always {
                    junit 'test-reports/results.xml'
                }
                input message: 'Lanjutkan ke tahap Deploy? (Click "Proceed" to continue)'
            }
        }
    }
    stage('Deploy') {
        try {
            env.VOLUME = "${pwd()}/sources:/src"
            env.IMAGE = "cdrx/pyinstaller-linux:python2"
            dir(env.BUILD_ID) {
                unstash(name: 'compiled-results')
                sh "docker run --rm -v ${env.VOLUME} ${env.IMAGE} 'pyinstaller -F add2vals.py'"
            }
            archiveArtifacts "sources/dist/add2vals"
            sh "docker run --rm -v ${env.VOLUME} ${env.IMAGE} 'rm -rf build dist'"
        } catch (Exception e) {
            echo 'Error: ' + e.toString()
        }
    }
}
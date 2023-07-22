node {
     withDockerContainer('python:2-alpine') {
        stage('Build') {
            checkout scm
            sh 'pwd'
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*') 
        }
    }
    withDockerContainer('qnib/pytest') {
        stage('Test') {
            checkout scm
            sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            junit 'test-reports/results.xml'
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
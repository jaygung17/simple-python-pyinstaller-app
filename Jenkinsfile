node{
    stage('Build') {
        docker.image('python:2-alpine').inside{
	    checkout scm
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*') 
        }
    }
    try{
        stage('Test'){
            docker.image('qnib/pytest').inside{
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        }
    }
    catch (e){
        throw e
    }
    finally{
        docker.image('qnib/pytest').inside{
            junit 'test-reports/results.xml'
        }
    }
    try{
        stage('Deploy'){
            env.VOLUME = '$(pwd)/sources:/src'
            env.IMAGE = 'cdrx/pyinstaller-linux:python2'
            dir(path: env.BUILD_ID){
                unstash(name: 'compiled-results')
                sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
                }
            }
            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
    }
    catch(e){
        throw e
    }
}

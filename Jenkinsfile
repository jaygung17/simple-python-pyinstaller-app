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
    stage('Manual Approval'){
        input message: "Lanjutkan ke tahap Deploy?"
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
        def remote_server = "103.175.219.135"
        withCredentials([sshUserPrivateKey(credentialsId: 'local-vps', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USERNAME')]) {
            sh "scp -o \"StrictHostKeyChecking=no\" -i \${SSH_KEY} build/ ${SSH_USERNAME}@${remote_server}:pydev/"
            sh "ssh -o \"StrictHostKeyChecking=no\" -i \${SSH_KEY} ${SSH_USERNAME}@${remote_server} '~/pydev/add2vals 3 2'"
        }
            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
	    sleep 60
    }
    catch(e){
        throw e
    }
}

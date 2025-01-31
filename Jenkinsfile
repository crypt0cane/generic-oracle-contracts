@Library('global-jenkins-library@2.0.0') _

node('docker') {
    buildInfo = null

    stage('Build info') {
        buildInfo = getBuildInfo()
    }

    docker.image('node:16-alpine').inside {
        stage('Test') {
            checkout scm
            sh '''
            npm ci
            npx hardhat typechain
            npx hardhat coverage
            '''
            archiveArtifacts artifacts: 'coverage/'
        }

        if (buildInfo.versionNoPrefix != '') {
            stage('Publish') {
                sh 'npm version ' + buildInfo.versionNoPrefix + ' --allow-same-version'
                isPublished = false
                try {
                    withCredentials([string(credentialsId: 'NPM_TOKEN', variable: 'AUTH_TOKEN')]) {
                        sh '''
                        echo "//registry.npmjs.org/:_authToken=$AUTH_TOKEN" > ~/.npmrc
                        npm publish --access public
                        '''
                        isPublished = true
                    }
                } finally {
                    sh 'rm ~/.npmrc'
                    if(!isPublished) {
                        error 'Failed to publish to NPM.'
                    }
                }
            }
        }
    }
}

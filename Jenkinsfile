node('raspi') {
    sh 'rm -f *.deb'
    sh 'sudo dpkg --remove libstromx libstromx-dev python-stromx'
    git 'https://github.com/uboot/stromx-pipeline.git'
    sh 'git submodule update --init --recursive'
    sha = sh script: 'git rev-parse --short HEAD',  returnStdout: true
    
    stage('stromx') {
        dir('stromx') {
            sh 'git checkout debian/changelog'
            sh 'git checkout debian/control'
            withEnv(['SHA=' + sha.replaceAll("\\s","") + "-" + env.BUILD_ID]) {
                sh 'sed -i.bak -r "s/\\(([0-9].[0-9].[0-9])\\)/(\\1-${SHA})/" debian/changelog'
            }
            sh 'sed -i.bak s/libboost-all-dev,// debian/control'
            sh 'sed -i.bak s/doxygen,// debian/control'
            sh 'sed -i.bak /stromx-doc/,+5d debian/control'
            sh 'dpkg-buildpackage -j4 -us -uc'
        }
        sh 'sudo dpkg --install *.deb'
    }
    
    stage('stromx-opencv') {
        dir('stromx-opencv') {
            sh 'git checkout debian/changelog'
            withEnv(['SHA=' + sha.replaceAll("\\s","") + "-" + env.BUILD_ID]) {
                sh 'sed -i.bak -r "s/\\(([0-9].[0-9].[0-9])\\)/(\\1-${SHA})/" debian/changelog'
            }
            sh 'dpkg-buildpackage -j4 -us -uc'
        }
    }
    
    stage('stromx-zbar') {
        dir('stromx-zbar') {
            sh 'git checkout debian/changelog'
            withEnv(['SHA=' + sha.replaceAll("\\s","") + "-" + env.BUILD_ID]) {
                sh 'sed -i.bak -r "s/\\(([0-9].[0-9].[0-9])\\)/(\\1-${SHA})/" debian/changelog'
            }
            sh 'dpkg-buildpackage -j4 -us -uc'
        }
    }
    
    stash includes: '*.deb', name: 'stromx'
}

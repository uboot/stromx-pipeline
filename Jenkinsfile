node('raspi-build') {
    sh 'rm -f *.deb *.tar.gz *.changes *.dsc'
    sh 'sudo dpkg --remove libstromx libstromx-dev python-stromx'
    git 'https://github.com/uboot/stromx-pipeline.git'
    sh 'git submodule update --init --recursive'
    sha = sh script: 'git rev-parse --short HEAD',  returnStdout: true

    stage('stromx') {
        dir('stromx') {
            patchVersion(sha)
            sh 'git checkout debian/control'
            sh 'sed -i.bak s/libboost-all-dev,// debian/control'
            sh 'sed -i.bak s/doxygen,// debian/control'
            sh 'sed -i.bak /stromx-doc/,+5d debian/control'
            sh 'dpkg-buildpackage -j4 -us -uc'
        }
        sh 'sudo dpkg --install *.deb'
    }

    stage('stromx-opencv') {
        dir('stromx-opencv') {
            patchVersion(sha)
            sh 'dpkg-buildpackage -j4 -us -uc'
        }
    }

    stage('stromx-zbar') {
        dir('stromx-zbar') {
            patchVersion(sha)
            sh 'dpkg-buildpackage -j4 -us -uc'
        }
    }

    stage('stromx-raspi') {
        dir('stromx-raspi') {
            patchVersion(sha)
            sh 'dpkg-buildpackage -j4 -us -uc'
        }
    }

    stage('stromx-web') {
        dir('stromx-web') {
            patchVersion(sha)
            sh 'dpkg-buildpackage -j4 -us -uc'
        }
    }

    stash includes: '*.deb, stromx.conf', name: 'stromx'
}

node('raspi-deploy') {
    stage('deploy') {
        sh returnStatus: true, script: 'stromx-server stop'
        sh 'sudo dpkg --remove libstromx python-stromx libstromx-opencv libstromx-zbar libstromx-raspi stromx-web'
        sh 'rm -f *.deb'
        unstash name: 'stromx'
        sh 'sudo dpkg --install libstromx_*.deb python-stromx_*.deb libstromx-opencv_*.deb libstromx-zbar_*.deb libstromx-raspi_*.deb stromx-web_*.deb'
        sh 'sudo cp stromx.conf /etc/stromx'
        sh 'stromx-server start'
    }
}

def patchVersion(String sha) {
    sh 'git checkout debian/changelog'
    withEnv(['SHA=' + sha.replaceAll("\\s","") + "-" + env.BUILD_ID]) {
        sh """sed -i.bak -r 's/\\(([0-9].[0-9].[0-9])\\)/(\\1-${SHA})/'\
              debian/changelog"""
    }
}

@Library("yooture")
import yooture.jenkins.yooture.YooBuild

def yooBuild = new YooBuild(this)

pipeline {
  agent { label 'lxc-fedora25' }

  tools {
    maven 'MVN_352'
    jdk "Oracle JDK 1.8 (latest)"
  }
  
  environment {
    // hack because path to the custom tool is not correctly added to PATH
    JX_PATH = tool name: 'jx-release-version', type: 'com.cloudbees.jenkins.plugins.customtools.CustomTool'
    PATH = "$JX_PATH:$PATH"
  }  

  options {
    // General Jenkins job properties
    buildDiscarder(logRotator(numToKeepStr:'10'))
  }

  stages { 

    stage('build Snapshot') {
      when { not { branch 'master' } }
      steps {
        timeout(time: 20, unit: 'MINUTES') {
          withMaven(maven: 'MVN_352',options: [artifactsPublisher(disabled: true)]) {
            sh 'mvn clean install -Dmaven.test.failure.ignore'
          }
        }
      }
    }

    stage('Build Release') {
      when { branch 'master' }
      steps {
          script {
	          def releasedArtifact = yooBuild.mvnReleaseMasterBranch(".", { })
	          hipchatSend color: 'GREEN', room: 'yooture', notify: true, message: "@all new version to maven repository released: ${releasedArtifact}"
          }
      }
    }      

  }
  post {
    always {
      cleanWs()
    }
    unstable {
      hipchatSend color: 'YELLOW', room: 'yooture', notify: true, message: "@all please fix!!! ${currentBuild.displayName}: ${currentBuild.result} - ${currentBuild.absoluteUrl}"
    }
    failure {
      hipchatSend color: 'RED', room: 'yooture', notify: true, message: "@all please fix!!! ${currentBuild.displayName}: ${currentBuild.result} - ${currentBuild.absoluteUrl}"
    }
  }
}
pipeline {
  agent any

  options {
    buildDiscarder(logRotator(numToKeepStr:'15'))
  }

  tools { 
    maven 'apache-maven-latest'
    jdk 'oracle-jdk8-latest'
  }
  
  // https://jenkins.io/doc/book/pipeline/syntax/#triggers
  triggers {
    pollSCM('H/5 * * * *')
  }
  
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Gradle Build') {
      steps {
        sh '''
          ./gradlew clean cleanGenerateXtext build createLocalMavenRepo \
            -PcompileXtend=true \
            -PJENKINS_URL=$JENKINS_URL \
            -PignoreTestFailures=true \
            --refresh-dependencies \
            --continue
        '''
        step([$class: 'JUnitResultArchiver', testResults: '**/build/test-results/test/*.xml'])
      }
    }
    
    stage('Maven Build') {
      steps {
        dir('.m2/repository/org/eclipse/xtext') { deleteDir() }
        dir('.m2/repository/org/eclipse/xtend') { deleteDir() }
        sh '''
          mvn \
            -f releng \
            --batch-mode \
            --update-snapshots \
            -Dmaven.repo.local=.m2/repository \
            -DJENKINS_URL=$JENKINS_URL \
            -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=warn \
            clean install
        '''
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: 'build/**'
    }
  }
}
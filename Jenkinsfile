pipeline {
  agent any
  stages {
    stage('product1') {
      parallel {
        stage('product1') {
          steps {
            withMaven()
            echo 'ccccccccccc'
          }
        }
        stage('f1') {
          steps {
            echo 'aaaaaaaaaaaa'
          }
        }
        stage('') {
          steps {
            echo 'bbbbbbbbbbbb'
          }
        }
      }
    }
    stage('phase2') {
      parallel {
        stage('phase2') {
          steps {
            echo '2aaaa'
          }
        }
        stage('s2') {
          steps {
            echo '2 bbbbbbb'
          }
        }
      }
    }
    stage('phase3') {
      steps {
        echo '3 aaaaa'
      }
    }
  }
}
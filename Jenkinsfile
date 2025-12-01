pipeline {
  agent any
  options {
    // keep a few builds to save space
    buildDiscarder(logRotator(numToKeepStr: '10'))
    ansiColor('xterm')
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Decide file to build') {
      steps {
        script {
          // Map branch -> file (adjust names if your classes differ)
          def fileMap = [
            'feature-login'      : 'login.java',
            'feature-result'     : 'result.java',
            'feature-attendance' : 'attendance.java',
            'main'               : 'student.java' // optional: default on main
          ]

          // Choose file based on branch name; fall back to any single .java if absent
          BUILD_FILE = fileMap[env.BRANCH_NAME] ?: shReturn("ls -1 *.java | head -n1")
          echo "Branch: ${env.BRANCH_NAME} -> building file: ${BUILD_FILE}"
        }
      }
    }

    stage('Compile') {
      steps {
        script {
          // make directories
          if (isUnix()) {
            sh 'mkdir -p out artifacts'
            // compile into out
            sh "javac -d out ${BUILD_FILE}"
          } else {
            bat 'if not exist out mkdir out'
            bat 'if not exist artifacts mkdir artifacts'
            bat "javac -d out ${BUILD_FILE}"
          }
        }
      }
    }

    stage('Run') {
      steps {
        script {
          // derive class name (strip .java)
          def className = BUILD_FILE.tokenize('/')[-1].minus('.java')
          if (isUnix()) {
            sh "java -cp out ${className}"
          } else {
            bat "java -cp out ${className}"
          }
        }
      }
    }

    stage('Collect artifacts') {
      steps {
        script {
          if (isUnix()) {
            sh '''
              mv out/*.class artifacts/ 2>/dev/null || true
              echo "Build from ${BRANCH_NAME}" > artifacts/artifact.txt
              ls -la artifacts || true
            '''
          } else {
            bat '''
              move /Y out\\*.class artifacts >nul 2>&1 || echo no classes moved
              echo Build from %BRANCH_NAME% > artifacts\\artifact.txt
              dir artifacts
            '''
          }
        }
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: 'artifacts/**', fingerprint: true
      echo "Build and archive successful for branch ${env.BRANCH_NAME}"
    }
    failure {
      echo "Build failed for branch ${env.BRANCH_NAME}"
    }
  }
}

// Helper function for sh fallback when BUILD_FILE not set (Groovy + shell)
def shReturn(cmd) {
  def output = ''
  try {
    output = sh(script: cmd, returnStdout: true).trim()
  } catch (err) {
    // if shell not available or fails, try to read workspace listing via groovy
    output = ''
  }
  return output
}


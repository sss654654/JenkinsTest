pipeline {
  agent any
  options { timestamps() }
  triggers { githubPush() }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Setup venv') {
      steps {
        sh '''
          python3 -m venv .venv
          . .venv/bin/activate
          python --version
          pip install -U pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          . .venv/bin/activate
          mkdir -p reports
          pytest -q \
            --junitxml=reports/junit.xml \
            --cov=. --cov-report=xml:reports/coverage.xml || EXIT=$?
          # pytest 종료코드를 저장했다가, 파일 업로드 위해 0으로 종료하지 않음
          exit ${EXIT:-0}
        '''
      }
      post {
        always {
          // 테스트 리포트와 커버리지 리포트 업로드
          junit 'reports/junit.xml'
          archiveArtifacts artifacts: 'reports/**', fingerprint: true
        }
        // 테스트 실패 시 빌드도 실패로 표시
        unsuccessful {
          script { currentBuild.result = 'FAILURE' }
        }
      }
    }
  }
}

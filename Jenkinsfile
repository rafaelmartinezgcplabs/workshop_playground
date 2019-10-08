pipeline {
  agent {
    docker {
      alwaysPull true
      image "bitsydarel/flutter-ci:latest"
    }
  }

  stages {
    stage("Initialize CI/CD") {
      steps {
        sh 'alias fastlane="bundle exec fastlane"'
      }
    }
    stage("Code style check") {
      when {
        expression {
          env.CHANGE_ID != null
        }
      }
      steps {
        withCredentials([string(credentialsId: 'github_token', variable: 'GITHUB_API_TOKEN')]) {
          sh "dbstyleguidechecker -s analysis_options.yaml -f -g $GIT_URL -p $CHANGE_ID -t $GITHUB_API_TOKEN"
        }
      }
    }
    stage("Unit tests") {
      steps {
        sh "flutter test --coverage --coverage-path lcov.info"
      }
    }
    stage("Development Integration tests") {
      when {
        expression {
          env.CHANGE_ID != null
        }
      }
      steps {
        sh "launch_android_emulator.sh"
        sh "flutter drive --debug --target=test_driver/app.dart"
      }
    }
    stage("Release Integration tests") {
      when {
        expression {
          env.CHANGE_ID == null
        }
      }
      steps {
        sh "launch_android_emulator.sh"
        sh "flutter drive --release --target=test_driver/app.dart"
      }
    }
    stage("Build For QA For Android") {
      when {
        expression {
          env.CHANGE_ID != null
        }
      }
      steps {
        sh "cd android && bundle update && cd ../"
        sh "flutter build apk --debug"
        sh "cd android && echo 'fastlane <name of the lane>'"
      }
    }
    stage("Build For Debug For iOS") {
      when {
        expression {
          env.CHANGE_ID != null
        }
      }
      steps {
        sh "cd ios && bundle update && cd ../"
        sh "flutter build ios --debug --no-codesign"
        sh "cd ios && echo 'fastlane <name of the lane>'"
      }
    }
    stage("Build For Release For Android") {
      when {
        expression {
          env.CHANGE_ID == null
        }
      }
      steps {
        sh "cd android && bundle update && cd ../"
        sh "flutter build apk --release"
        sh "cd android && echo 'fastlane <name of the lane>'"
      }
    }
    stage("Build For Release For iOS") {
      when {
        expression {
          env.CHANGE_ID == null
        }
      }
      steps {
        sh "cd ios && bundle update && cd ../"
        sh "flutter build ios --release --no-codesign"
        sh "cd ios && echo 'fastlane <name of the lane>'"
      }
    }
  }


  post {
    success {
     script {
       // if we are in a PR
       if (env.CHANGE_ID) {
         publishCoverageGithub(filepath:'lcov.info', coverageType: 'lcov', comparisonOption: [ value: 'optionFixedCoverage', fixedCoverage: '0.75' ], coverageRateType: 'Overall')
       }
     }
    }
    cleanup {
      cleanWs(notFailBuild: true, cleanWhenFailure: false, cleanWhenNotBuilt: false, cleanWhenUnstable: false)
    }
  }
}
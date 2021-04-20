pipeline {
  agent {
    docker {
      alwaysPull true
      image 'bitsydarel/flutter-ci:latest'
      args '-u root'
    }
  }
  stages {
    stage("Initialize CI/CD") {
      steps {
        sh 'alias fastlane="bundle exec fastlane"'
        sh 'flutter -v precache'
        sh 'flutter doctor -v'
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
    stage("Integration Tests") {
      parallel {
        stage("Development Integration tests") {
          when {
            expression {
              env.CHANGE_ID != null
            }
          }
          steps {
            sh "launch_android_emulator.sh"
            sh "flutter -v drive --debug --target=test_driver/app.dart"
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
            sh "flutter -v drive --debug --target=test_driver/app.dart"
          }
        }
      }
    }
    stage("Release QA") {
      when {
        expression {
          env.CHANGE_ID != null
        }
      }
      parallel {
        stage("Build For QA For Android") {
          steps {
            sh "cd android && bundle install --path vendor/bundle && cd ../"
            sh "flutter -v build apk --debug"
            sh "cd android && echo 'fastlane <name of the lane>'"
          }
        }
        stage("Build For Debug For iOS") {
          steps {
            sh "cd ios && bundle install --path vendor/bundle && cd ../"
            sh "flutter -v build ios --debug --no-codesign"
            sh "cd ios && echo 'fastlane <name of the lane>'"
          }
        }
      }
    }
    stage("Release Live") {
      when {
        expression {
          env.CHANGE_ID == null
        }
      }
      parallel {
        stage("Build For Release For Android") {
          steps {
            sh "cd android && bundle install --path vendor/bundle && cd ../"
            sh "flutter -v build apk --release"
            sh "cd android && echo 'fastlane <name of the lane>'"
          }
        }
        stage("Build For Release For iOS") {
          steps {
            sh "cd ios && bundle install --path vendor/bundle && cd ../"
            sh "flutter -v build ios --release --no-codesign"
            sh "cd ios && echo 'fastlane <name of the lane>'"
          }
        }
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
      cleanWs(notFailBuild: true, cleanWhenFailure: true, cleanWhenNotBuilt: false, cleanWhenUnstable: false)
    }
  }
}

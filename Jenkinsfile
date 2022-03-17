pipeline {
  agent {
    docker { image 'rust:latest' }
  }

  options {
    timeout(time: 60, unit: 'MINUTES')
    ansiColor('xterm')
    disableConcurrentBuilds(abortPrevious: true)
    buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '5', numToKeepStr: '5')
  }

  environment {
    GITHUB_ORGANIZATION = "halkeye"
    GITHUB_REPO = "typos-json-to-checkstyle"
    PROJECT_NAME = "typos-checkstyle"
  }

  stages {
    stage('Check') {
      steps {
        sh '''
          cargo check --message-format json | tee cargocheck.json || true
        '''
      }
      post {
        always {
          recordIssues(tools: [cargo(pattern: 'cargocheck.json')])
        }
      }
    }

    stage('Build') {
      steps {
        sh 'cargo build --locked --all-targets --release'
      }
    }

    stage('Test') {
      steps {
        sh 'cargo test'
      }
    }

    stage('Lint') {
      steps {
        sh '''
          rustup component add clippy
          cargo clippy --all-targets -- -D warnings
        '''
      }
    }

    stage('Format') {
      steps {
        sh '''
          rustup component add rustfmt
          cargo fmt -- --check
        '''
      }
    }

    stage('Doc') {
      steps {
        // Not sure what to do with this yet
        sh 'cargo doc'
      }
    }

    stage('Release Dry Run') {
      when { not { buildingTag() } }
      steps {
        sh '''
          cargo publish --dry-run
        '''
      }
    }

    stage('Create github release') {
      when { buildingTag() }
      environment {
        GITHUB = credentials('github-halkeye')
        GITHUB_TOKEN = "${env.GITHUB.split(":")[1]}"
      }
      steps {
        sh '''
          curl -qsL https://github.com/github-release/github-release/releases/download/v0.10.0/linux-amd64-github-release.bz2 | bzip2 -d > github-release && chmod 755 ./github-release
          curl -qsL https://github.com/taiki-e/parse-changelog/releases/download/v0.4.7/parse-changelog-x86_64-unknown-linux-musl.tar.gz | tar xvzf - parse-changelog

          # delete the existing release (if exists)
          ./github-release delete --user "${GITHUB_ORGANIZATION}" --repo "${GITHUB_REPO}" --tag "${TAG_NAME}" || true

          # Creating a new release in github
          ./github-release release --user "${GITHUB_ORGANIZATION}" --repo "${GITHUB_REPO}" --tag "${TAG_NAME}" --name "${TAG_NAME}" --description "$(./parse-changelog)"

          # Uploading the artifacts into github
          ./github-release upload --user "${GITHUB_ORGANIZATION}" --repo "${GITHUB_REPO}" --tag "${TAG_NAME}" --name "${PROJECT_NAME}-${TAG_NAME}-$(arch)" --file "target/release/${PROJECT_NAME}"
        '''
      }
    }

    stage('Upload binaries to github') {
      when { buildingTag() }
      environment {
        GITHUB = credentials('github-halkeye')
        GITHUB_TOKEN = "${env.GITHUB.split(":")[1]}"
      }
      steps {
        sh '''
          ./github-release upload --user "${GITHUB_ORGANIZATION}" --repo "${GITHUB_REPO}" --tag "${TAG_NAME}" --name "${PROJECT_NAME}-${TAG_NAME}-$(arch)" --file "target/release/${PROJECT_NAME}"
        '''
      }
    }

    stage('Release to cargo') {
      // when { branch 'main' }
      when { buildingTag() }
      environment {
        CARGO_REGISTRY_TOKEN = credentials('halkeye-crates')
      }
      steps {
        sh '''
          cargo login

          cargo publish
        '''
      }
    }
  }
}

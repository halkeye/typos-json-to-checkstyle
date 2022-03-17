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

    stage('Release') {
      // when { branch 'main' }
      when { buildingTag() }
      environment {
        CARGO_REGISTRY_TOKEN = credentials('halkeye-crates')
        GITHUB = credentials('github-halkeye')
        GITHUB_TOKEN = "${env.GITHUB.split(":")[1]}"
      }
      steps {
        sh '''
          cargo login

          curl -qsL https://github.com/github-release/github-release/releases/download/v0.10.0/linux-amd64-github-release.bz2 | bzip2 -d > github-release && chmod 755 ./github-release

          # delete the existing release (if exists)
          ./github-release delete --user "${GITHUB_ORGANIZATION}" --repo "${GITHUB_REPO}" --tag "${TAG_NAME}" || true

          # Creating a new release in github
          ./github-release release --user "${GITHUB_ORGANIZATION}" --repo "${GITHUB_REPO}" --tag "${TAG_NAME}" --name "${TAG_NAME}"

          # Uploading the artifacts into github
          ./github-release upload --user "${GITHUB_ORGANIZATION}" --repo "${GITHUB_REPO}" --tag "${TAG_NAME}" --name "${PROJECT_NAME}-${TAG_NAME}-$(arch)" --file "target/release/${PROJECT_NAME}"

          cargo publish
        '''
      }
    }
  }
}

pipeline {
  agent {
    docker { image 'rust:latest' }
  }
  stages {
    stage('Build') {
      steps {
        sh 'cargo build'
        archiveArtifacts 'target/debug/typos-checkstyle'
      }
    }

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
      steps {
        sh '''
          cargo publish --dry-run
        '''
      }
    }
  }
}

pipeline{
    agent any

    tools {
        nodejs 'nodejs-22-6-0'
    }

    stages {
      stage('VM node version'){
        steps {
              sh '''
                  node -v
                  npm -v
              '''
        }

      }
    }
  }

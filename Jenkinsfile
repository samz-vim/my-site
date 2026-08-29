pipeline {
    agent any

    stages {
        stage('build') {
            steps {
                sh 'echo "Hello World"'
                sh " whoami"
                sh " docker --version"
                sh " docker build -t my-site ./web"
            }
        }
    }
         stage('Run') {
            steps {
                sh 'docker run -d -p 3000:3000 my-site'
            }
         }
}

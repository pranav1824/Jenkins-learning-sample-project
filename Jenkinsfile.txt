
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Code downloaded from GitHub'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    test -f index.html
                    test -f styles/style.css
                    test -d images
                '''
                echo 'Website files verified'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    grep -qi '<html' index.html
                    grep -qi '</html>' index.html
                    grep -q 'styles/style.css' index.html
                '''
                echo 'HTML and CSS checks passed'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sudo rsync -av --delete \
                        --exclude='Jenkinsfile' \
                        --exclude='.git/' \
                        ./ /var/www/html/
                '''
                echo 'Website deployed to Nginx'
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}

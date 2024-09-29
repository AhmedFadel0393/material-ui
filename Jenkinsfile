pipeline {
    agent any

    environment {
        // Define variables for the build process
        GITHUB_REPO = "https://github.com/AhmedFadel0393/material-ui.git"
        RECIPIENTS = "developer@example.com" // Replace with actual email
        GIT_USER_NAME = "AhmedFadel0393"
        GIT_USER_EMAIL = "ahmed.fadel0393@gmail.com"
    }

    stages {
        stage('Increase Git Buffer Size') {
            steps {
                // Increase the Git buffer size to 500MB to handle large fetches
                sh 'git config --global http.postBuffer 524288000'
            }
        }

        stage('Checkout Code') {
            steps {
                // Checkout the code from the GitHub repo
                git branch: 'main', credentialsId: 'github-token', url: "${GITHUB_REPO}"
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install all required npm dependencies
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                // Run the Jest tests to ensure all components work as expected
                sh 'npm test'
            }
        }

        stage('Build Project') {
            steps {
                // Build the project to prepare for deployment
                sh 'npm run build'
            }
        }

        stage('Version and Tag Release') {
            steps {
                script {
                    // Use semantic-release or manually version and tag the release
                    def currentVersion = sh(script: "npm version patch --no-git-tag-version", returnStdout: true).trim()
                    sh "git tag -a v${currentVersion} -m 'Release version ${currentVersion}'"
                    sh "git push origin v${currentVersion}"
                }
            }
        }

        stage('Push to GitHub') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    // Configure Git user details using environment variables and push changes back to GitHub
                    sh '''
                        git config user.email "${GIT_USER_EMAIL}"
                        git config user.name "${GIT_USER_NAME}"
                        git add .
                        git commit -m "Automated Build & Test - New Version"
                        git push https://$GITHUB_TOKEN@github.com/AhmedFadel0393/material-ui.git
                    '''
                }
            }
        }
    }

    post {
        success {
            // Send success email notification
            emailext (
                subject: "Build Successful: UI Library - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Good news!</p>
                         <p>The build for <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> was successful.</p>
                         <p>Check the latest version at: <a href="${GITHUB_REPO}">GitHub Repository</a></p>
                         <p>Jenkins Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>""",
                to: "${RECIPIENTS}",
                mimeType: 'text/html'
            )
        }
        failure {
            // Send failure email notification
            emailext (
                subject: "Build Failed: UI Library - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Attention Required!</p>
                         <p>The build for <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> has failed.</p>
                         <p>Please check the logs to identify and fix the issue.</p>
                         <p>Jenkins Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>""",
                to: "${RECIPIENTS}",
                mimeType: 'text/html'
            )
        }
    }
}

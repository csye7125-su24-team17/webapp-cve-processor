pipeline {
    agent any
    environment {
        TARGET_BRANCH = 'main'
    }
    stages {
        stage('Lint Commit Messages') {
            steps {
                script {
                    // Using withCredentials to securely provide GitHub credentials
                    withCredentials([usernamePassword(credentialsId: 'github-pat', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                        // Correctly setting up Git to use credentials
                        sh 'git config credential.helper "!f() { echo username=\\$GIT_USERNAME; echo password=\\$GIT_PASSWORD; }; f"'
                        
                        // Fetching the target branch to compare differences
                        sh "git fetch --no-tags origin +refs/heads/${env.TARGET_BRANCH}:refs/remotes/origin/${env.TARGET_BRANCH}"

                        // Extracting only the last commit message
                        def lastCommitMsg = sh(script: "git log -1 HEAD --pretty=format:'%s'", returnStdout: true).trim()

                        // Check the last commit message against the Conventional Commits format
                        if (!lastCommitMsg.matches("^(feat|fix|docs|style|refactor|perf|test|chore|revert|ci|build)(!)?(\\(\\S+\\))?\\: .+")) {
                            echo "The last commit message does not follow Conventional Commits format:"
                            echo " - ${lastCommitMsg}"
                            error "The last commit message is not in the Conventional Commits format. PR cannot be merged."
                        }
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'The last commit message follows the Conventional Commits format.'
        }
        failure {
            echo 'Commit message validation failed. Please ensure the last commit follows the Conventional Commits format.'
        }
    }
}

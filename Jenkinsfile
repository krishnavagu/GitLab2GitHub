pipeline {
    agent any

    environment {
        GITLAB_CREDENTIALS = 'gitlab-credentials-id'  // Replace with your GitLab credentials ID
        GITHUB_CREDENTIALS = 'github-credentials-id'  // Replace with your GitHub credentials ID
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Input Parameters') {
            steps {
                script {
                    GITLAB_REPOS_LIST = ['https://gitlab.com/cicd-migration/autosys_cimigration.git']
                    GITLAB_BRANCHES_LIST = ['F-Autosys_consolidate']
                    GITHUB_ORG = 'VenkataSalesforce'

                    if (GITLAB_REPOS_LIST.size() != GITLAB_BRANCHES_LIST.size()) {
                        error "Number of repositories does not match the number of branches."
                    }
                }
            }
        }

        stage('Clone from GitLab and Push to GitHub') {
            steps {
                script {
                    GITLAB_REPOS_LIST.eachWithIndex { repo, index ->
                        branch = GITLAB_BRANCHES_LIST[index]
                        echo "Cloning from GitLab: ${repo} (Branch: ${branch})"

                        dir("autosys_cimigration") {
                            // Clone from GitLab
                            checkout([$class: 'GitSCM',
                                      branches: [[name: "*/${branch}"]],
                                      userRemoteConfigs: [[url: repo, credentialsId: GITLAB_CREDENTIALS]]])

                            // Clean up Git credentials helper
                            sh """
                                git config --global --unset-all credential.helper
                                git config --global credential.helper store
                            """

                            // Set GitHub remote URL using GitHub credentials
                            sh """
                                git remote set-url origin https://x-access-token:${GITHUB_CREDENTIALS}@github.com/${GITHUB_ORG}/${repo}.git
                            """

                            // Push to GitHub
                            sh """
                                git push --set-upstream origin ${branch}
                                git push --tags
                            """
                        }
                    }
                }
            }
        }

        // Add a 2-minute wait (120 seconds) here:
        stage('Wait for 2 Minutes') {
            steps {
                echo "Waiting for 2 minutes..."
                sleep time: 2, unit: 'MINUTES'  // This will pause the pipeline for 2 minutes (120 seconds)
            }
        }

        stage('Post-Migration Verification') {
            steps {
                script {
                    echo "Post-migration verification..."
                    // Add verification steps as needed
                    sh 'git log --oneline'
                }
            }
        }
    }

    post {
        success {
            echo 'Migration from GitLab to GitHub completed successfully!'
        }
        failure {
            echo 'Migration failed due to credential or other errors.'
        }
    }
}

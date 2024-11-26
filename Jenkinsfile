pipeline {
    agent any

    parameters {
        string(name: 'GITLAB_REPOS', defaultValue: 'https://gitlab.com/cicd-migration/autosys_cimigration.git', description: 'List of GitLab repositories to migrate (comma-separated).')
        string(name: 'GITLAB_BRANCHES', defaultValue: 'F-Autosys_consolidate', description: 'List of GitLab branches to migrate (comma-separated).')
        string(name: 'GITHUB_ORG', defaultValue: 'VenkataSalesforce', description: 'GitHub organization URL to push the code into (e.g., VenkataSalesforce).')
        string(name: 'GITHUB_TOKEN', description: 'GitHub Personal Access Token for authentication') // Token input for GitHub
    }

    environment {
        GITLAB_CREDENTIALS = 'gitlab-credentials-id'
        GITHUB_TOKEN = credentials('github-credentials-id') // This will use the secret token stored in Jenkins
    }

    stages {
        stage('Input Parameters') {
            steps {
                script {
                    GITLAB_REPOS_LIST = params.GITLAB_REPOS.split(',')
                    GITLAB_BRANCHES_LIST = params.GITLAB_BRANCHES.split(',')
                    GITHUB_ORG = params.GITHUB_ORG

                    if (GITLAB_REPOS_LIST.size() != GITLAB_BRANCHES_LIST.size()) {
                        error "The number of repositories does not match the number of branches."
                    }
                }
            }
        }

        stage('Clone from GitLab and Push to GitHub') {
            steps {
                script {
                    GITLAB_REPOS_LIST.eachWithIndex { repo, index ->
                        branch = GITLAB_BRANCHES_LIST[index]
                        echo "Cloning from GitLab repository: ${repo} (Branch: ${branch})"

                        // Clone the repository from GitLab
                        dir("autosys_cimigration") {
                            checkout([$class: 'GitSCM', 
                                      branches: [[name: "*/${branch}"]], 
                                      userRemoteConfigs: [[url: repo, credentialsId: GITLAB_CREDENTIALS]]])

                            // Verify that the repository is cloned correctly
                            echo "Verifying that the Git repository is properly initialized..."
                            sh 'ls -la'  // List all files including hidden files
                            sh 'ls -la .git'  // Check if .git directory is present

                            // Ensure the GitHub organization repository URL
                            repoName = repo.tokenize('/').last().replace('.git', '')  // Extract repo name from GitLab URL
                            GITHUB_REPO_URL = "${GITHUB_ORG}/${repoName}"

                            // Try to checkout the branch in GitHub repo. If it exists, switch to it. If not, create it.
                            echo "Checking out branch ${branch} in GitHub repository..."
                            sh """
                                git fetch origin
                                git checkout ${branch} || git checkout -b ${branch}
                            """

                            // Set the correct GitHub repository URL (Token-based authentication)
                            echo "Setting GitHub repository URL: ${GITHUB_REPO_URL}"
                           sh """
                            git config --global credential.helper 'store --file=/tmp/git-credentials'
                            echo https://x-access-token:${GITHUB_TOKEN}@github.com > /tmp/git-credentials
                            git remote set-url origin https://github.com/${GITHUB_REPO_URL}.git
                            """


                            // Push the GitLab repository to the GitHub organization repository
                            echo "Pushing changes to GitHub repository..."
                            sh """
                                git push --set-upstream origin ${branch}
                                git push --tags
                            """
                        }
                    }
                }
            }
        }

        stage('Post-Migration Verification') {
            steps {
                script {
                    // After pushing, we need to ensure the Git repo context is available for verification
                    echo "Verifying that we are still in a valid Git repository..."

                    // Check the working directory and its contents to confirm that we are in a Git repository
                    echo "Checking current directory..."
                    sh 'pwd'
                    sh 'ls -la'

                    // Ensure we're in the correct directory where `.git` exists
                    echo "Listing .git directory contents..."
                    sh 'ls -la autosys_cimigration/.git'  // Explicitly point to the Git repo

                    // Now, verify the repository by running git log and git ls-tree commands
                    echo "Verifying the Git log and repository files..."
                    sh '''
                        cd autosys_cimigration
                        git log --oneline --decorate --graph
                        git ls-tree -r HEAD --name-only
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Repository migration from GitLab to GitHub organization completed successfully for all repositories!'
        }
        failure {
            echo 'Migration failed!'
        }
    }
}

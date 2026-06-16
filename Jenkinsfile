pipeline {
    agent any

    environment {
        GIT_USER = "jenkins-bot"
        GIT_EMAIL = "jenkins@local"
    }

    triggers {
        // Trigger on pushes to develop branch
        pollSCM('* * * * *') // or use GitHub webhook integration
    }

    parameters {
        string(name: 'TARGET_REF', defaultValue: 'develop', description: 'Branch or commit SHA to tag')
        string(name: 'TAG_DESCRIPTION', defaultValue: '', description: 'Tag description (manual only)')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the same repo that triggered the pipeline
                    checkout scm
        
                    // If a different branch/commit was requested, switch to it
                    if (params.TARGET_REF && params.TARGET_REF != env.GIT_BRANCH) {
                        sh """
                        git fetch --all --tags
                        git checkout ${params.TARGET_REF}
                        """
                    }
                }
            }
        }

        stage('Setup Git') {
            steps {
                sh """
                git config user.name "${GIT_USER}"
                git config user.email "${GIT_EMAIL}"
                """
            }
        }

        stage('Compute Next Version') {
            steps {
                script {
                    sh """
                    set -e
                    git fetch --tags --force

                    LAST_TAG=\$(git tag --list 'v*.*.*' --sort=-v:refname | head -n 1)

                    if [ -z "\$LAST_TAG" ]; then
                      NEW_TAG="v1.0.0"
                    else
                      VERSION=\${LAST_TAG#v}
                      IFS='.' read -r MAJOR MINOR PATCH <<< "\$VERSION"
                      COMMIT_MSG=\$(git log -1 --pretty=%B)

                      if echo "\$COMMIT_MSG" | grep -q "BREAKING CHANGE"; then
                        MAJOR=\$((MAJOR + 1)); MINOR=0; PATCH=0
                      elif echo "\$COMMIT_MSG" | grep -q "^feat:"; then
                        MINOR=\$((MINOR + 1)); PATCH=0
                      else
                        PATCH=\$((PATCH + 1))
                      fi

                      NEW_TAG="v\$MAJOR.\$MINOR.\$PATCH"
                    fi

                    echo "NEW_TAG=\$NEW_TAG" > version.txt
                    """
                    env.NEW_TAG = readFile('version.txt').trim().split('=')[1]
                    echo "Computed next version: ${env.NEW_TAG}"
                }
            }
        }

        stage('Resolve Description') {
            steps {
                script {
                    def desc = params.TAG_DESCRIPTION
                    if (!desc?.trim()) {
                        desc = "Manual release for ${params.TARGET_REF}"
                    }
                    env.TAG_DESC = desc
                    echo "Resolved description: ${env.TAG_DESC}"
                }
            }
        }

        stage('Create and Push Tag') {
            steps {
                sh """
                git tag -a "${env.NEW_TAG}" -m "${env.TAG_DESC}"
                git push origin "${env.NEW_TAG}"
                """
            }
        }
    }
}

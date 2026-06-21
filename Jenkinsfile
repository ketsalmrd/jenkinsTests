pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/dotnet/sdk:10.0'
            args '-u root:root'
        }
    }

    environment {
        PROJECT_NAME = 'jenkinsTests'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                sh 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build --configuration Release --no-restore'
            }
        }

        stage('Test') {
            steps {
                sh 'echo "No test project yet — skipping."'
                // sh 'dotnet test --configuration Release --no-build'
            }
        }

        stage('Publish') {
            steps {
                sh 'dotnet publish --configuration Release --no-build --output ./publish'
            }
        }

        stage('Package') {
            steps {
                script {
                    // --- Semver versioning from git tags + commit message ---
                    // Keywords (Spanish): "MAYOR" = major bump, "MINOR" = minor bump, else patch bump.

                    // Ensure zip is available in the container
                    sh 'apt-get update -qq && apt-get install -y -qq zip'

                    // Get latest tag (strip 'v' prefix if present)
                    def latestTag = sh(script: 'git tag --sort=-v:refname | head -1', returnStdout: true).trim()
                    def (major, minor, patch) = [0, 0, 0]

                    if (latestTag) {
                        def ver = latestTag.replaceAll('^v', '')
                        def parts = ver.split('\\.')
                        if (parts.size() == 3) {
                            major = parts[0] as int
                            minor = parts[1] as int
                            patch = parts[2] as int
                        }
                    }

                    // Get current commit message (first line)
                    def commitMsg = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()

                    if (commitMsg.startsWith('MAYOR')) {
                        major++
                        minor = 0
                        patch = 0
                    } else if (commitMsg.startsWith('MINOR')) {
                        minor++
                        patch = 0
                    } else {
                        patch++
                    }

                    def version = "${major}.${minor}.${patch}"
                    echo "Derived version: ${version} (from commit: ${commitMsg.take(50)})"

                    // Tag the commit
                    sh "git config user.name 'Jenkins CI'"
                    sh "git config user.email 'jenkins@${env.NODE_NAME}'"
                    sh "git tag -a \"v${version}\" -m \"Version ${version}\""
                    sh 'git push origin --tags'

                    // Build artifact name
                    def fullVersion = "${version}+build.${env.BUILD_NUMBER}"
                    def artifactName = "${PROJECT_NAME}-${fullVersion}.zip"

                    sh "zip -j \"${artifactName}\" ./publish/*"

                    archiveArtifacts artifacts: artifactName, fingerprint: true
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}

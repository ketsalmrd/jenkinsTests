def computeSemver() {
    // Keywords: "MAJOR" = major bump, "MINOR" = minor bump, else patch bump.

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

    if (commitMsg.startsWith('MAJOR')) {
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

    return version
}

def archiveBuild(String projectName, String version) {
    def artifactName = "${projectName}-${version}.tar.gz"

    sh "tar -czf \"/jenkins/artifacts/${artifactName}\" -C ./publish ."

    // Copy to workspace so archiveArtifacts can find it
    sh "cp \"/jenkins/artifacts/${artifactName}\" \"./${artifactName}\""
    archiveArtifacts artifacts: "${artifactName}", fingerprint: true
}

node {
    env.PROJECT_NAME = 'jenkinsTests'

    stage('Checkout') {
        checkout scm
        sh 'git config --global --add safe.directory "$(pwd)"'
    }

    docker.image('mcr.microsoft.com/dotnet/sdk:10.0').inside('-v /jenkins/artifacts:/jenkins/artifacts') {

        stage('Restore') {
            sh 'dotnet restore'
        }

        stage('Build') {
            sh 'dotnet build --configuration Release --no-restore'
        }

        stage('Test') {
            echo 'No test project yet — skipping.'
            // sh 'dotnet test --configuration Release --no-build'
        }

        stage('Publish') {
            sh 'dotnet publish --configuration Release --no-build --output ./publish'
        }

        stage('Package') {
            def version = computeSemver()

            // Tag the commit and push using GitHub token credential
            withCredentials([usernamePassword(
                credentialsId: 'github-token',
                usernameVariable: 'GIT_USER',
                passwordVariable: 'GIT_TOKEN'
            )]) {
                sh "git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/ketsalmrd/jenkinsTests.git"
                sh "git config user.name 'Jenkins CI'"
                sh "git config user.email 'jenkins@${env.NODE_NAME}'"
                sh "git tag -a \"v${version}\" -m \"Version ${version}\""
                sh 'git push origin --tags'
            }

            archiveBuild(env.PROJECT_NAME, version)
        }
    }
}

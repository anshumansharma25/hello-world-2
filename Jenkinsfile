// ═══════════════════════════════════════════════════════════════════════════
// Jenkinsfile — TechBuild Solutions / hello-world-2
// Repository: https://github.com/jagdishmodi/hello-world-2.git

// Pipeline: Checkout → Build → Test → Quality Analysis → Archive → Notify
// ═══════════════════════════════════════════════════════════════════════════
 
pipeline {
 
    // ── Docker agent for isolated, reproducible builds ─────────────────────
    agent {
        docker {
            image 'eclipse-temurin:17-jdk-alpine'
            args  '-v $HOME/.m2:/root/.m2'    // Cache Maven dependencies between builds
        }
    }
 
    // ── Environment variables ───────────────────────────────────────────────
    environment {
        APP_NAME     = 'hello-world-2'
        APP_VERSION  = "1.0.${env.BUILD_NUMBER}"
        MAVEN_OPTS   = '-Xmx1024m -XX:+TieredCompilation'
        SONAR_URL    = 'http://sonarqube:9000'
        ARTIFACT_DIR = 'target'
    }
 
    // ── Pipeline-wide options ───────────────────────────────────────────────
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timestamps()
        ansiColor('xterm')
    }
 
    // ── Build on push to any branch; deploy only from main ─────────────────
    triggers {
        githubPush()    // Requires GitHub plugin — responds to webhook events
    }
 
    // ══════════════════════════════════════════════════════════════════════
    stages {
 
        // ── STAGE 1: Checkout ─────────────────────────────────────────────
        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch: ${env.GIT_BRANCH} | Commit: ${env.GIT_COMMIT[0..7]}"
                echo "Branch: ${env.GIT_BRANCH}"
                echo "Commit: ${env.GIT_COMMIT}"
            }
        }
    }
}

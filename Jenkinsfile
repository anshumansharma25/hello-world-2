pipeline {

    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'
            args '-v $HOME/.m2:/root/.m2'
        }
    }

    environment {
        APP_NAME     = 'hello-world-2'
        APP_VERSION  = "1.0.${env.BUILD_NUMBER}"
        MAVEN_OPTS   = '-Xmx1024m -XX:+TieredCompilation'
        SONAR_URL    = 'http://sonarqube:9000'
        ARTIFACT_DIR = 'target'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timestamps()
        ansiColor('xterm')
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm

                echo "Branch: ${env.GIT_BRANCH}"
                echo "Commit: ${env.GIT_COMMIT[0..7]}"
            }
        }

        stage('Build') {
            steps {
                echo "Building ${env.APP_NAME} v${env.APP_VERSION}"

                sh 'java -version'
                sh 'mvn -version'
                sh 'mvn clean compile -B -Dmaven.test.skip=true'
            }

            post {
                success {
                    echo 'Compile successful — moving to Test stage.'
                }

                failure {
                    echo 'Compile FAILED — check pom.xml and source errors.'
                }
            }
        }
    }
}

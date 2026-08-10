pipeline {

    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'
            args '-u 111:113 -v $HOME/.m2:/root/.m2'
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
                // ── STAGE 3: Test ─────────────────────────────────────────────────
        stage('Test') {
            steps {
                sh 'mvn test -B'
            }
            post {
                always {
                    // Publish JUnit test results regardless of pass/fail
                    junit(testResults: 'target/surefire-reports/**/*.xml',
                          allowEmptyResults: false)
                }
                unstable {
                    echo 'WARNING: Tests failed — build marked UNSTABLE.'
                    // Enforce 80% pass rate:
                    script {
                        def results = currentBuild.rawBuild.getAction(
                            hudson.tasks.test.AbstractTestResultAction.class)
                        if (results) {
                            def passRate = (results.totalCount - results.failCount) /
                                           results.totalCount * 100
                            if (passRate < 80) {
                                error("Test pass rate ${passRate.round(1)}% is below 80% threshold!")
                            }
                        }
                    }
                }
            }
        }
                // ── STAGE 4: Quality Analysis ─────────────────────────────────────
        stage('Quality Analysis') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh """
                mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                  -Dsonar.projectKey=${env.APP_NAME} \
                  -Dsonar.projectName="TechBuild ${env.APP_NAME}" \
                  -Dsonar.projectVersion=${env.APP_VERSION} \
                  -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \
                  -B
            """
                }
            }
        }

                // ── STAGE 5: Quality Gate ─────────────────────────────────────────
        stage('Quality Gate') {
            agent none
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

                // ── STAGE 6: Package & Archive ────────────────────────────────────
        stage('Package & Archive') {
            steps {
                sh "mvn package -DskipTests -B -Drevision=${env.APP_VERSION}"
                archiveArtifacts(artifacts: 'target/*.war', fingerprint: true)
                echo "Artifact archived: ${env.APP_NAME}-${env.APP_VERSION}.war"
            }
        }

                // ── STAGE 7: Publish to Nexus (main branch only) ─────────────────
        stage('Publish Artifact') {
            when { branch 'master' }
            steps {
                nexusArtifactUploader(
                    nexusVersion:  'nexus3',
                    protocol:      'http',
                    nexusUrl:      '172.17.0.3:8081',
                    groupId:       'io.techbuild',
                    version:       env.APP_VERSION,
                    repository:    'techbuild-releases',
                    credentialsId: 'nexus-cred',
                    artifacts: [[
                        artifactId: env.APP_NAME,
                        classifier: '',
                        file:       "target/${env.APP_NAME}-${env.APP_VERSION}.war",
                        type:       'war'
                    ]]
                )
            }
        }
 
    }    // end stages
    post {
    success {
        echo "PIPELINE SUCCESS — ${env.APP_NAME} v${env.APP_VERSION}"

        slackSend(
            channel: '#ci-notifications',
            color: 'good',
            message: "BUILD PASSED: ${env.APP_NAME} v${env.APP_VERSION} | ${env.BUILD_URL}"
        )
    }

    failure {
        echo "PIPELINE FAILED — check logs at ${env.BUILD_URL}"

        slackSend(
            channel: '#ci-notifications',
            color: 'danger',
            message: "BUILD FAILED: ${env.APP_NAME} #${env.BUILD_NUMBER} | ${env.BUILD_URL}"
        )
    }

    unstable {
        slackSend(
            channel: '#ci-notifications',
            color: 'warning',
            message: "BUILD UNSTABLE: ${env.APP_NAME} #${env.BUILD_NUMBER} — test failures | ${env.BUILD_URL}"
        )
    }

    always {
        cleanWs()
    }
}
}

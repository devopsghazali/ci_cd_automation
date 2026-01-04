pipeline {

    agent { label 'agent-1' }

    tools {
        jdk 'java-17'
        maven 'maven-3'
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout Source Code") {
            steps {
                git branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/devopsghazali/ci_cd_automation.git'
            }
        }

        stage("Secret Scan (GitLeaks)") {
            steps {
                sh '''
                 gitleaks detect \
  --source . \
  --no-git \
  --verbose \
  --report-format json \
  --report-path gitleaks-report.json \
  --exit-code 0

                '''
            }
        }

        stage("Compile Code") {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage("Run Unit Tests") {
            steps {
                sh 'mvn test'
            }
        }

        // stage("Dependency Vulnerability Scan (OWASP)") {
        //     steps {
        //         sh '''
        //           mvn org.owasp:dependency-check-maven:check \
        //           -Dformat=HTML \
        //           -DfailBuildOnCVSS=7
        //         '''
        //     }
        // }

        stage("Static Code Analysis (SonarQube)") {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                      mvn sonar:sonar \
                      -Dsonar.projectKey=register-app \
                      -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Package Artifact") {
            steps {
                sh 'mvn package'
            }
        }
    }
}

pipeline {

    agent { label 'agent-1' }

    tools {
        jdk 'java-17'
        maven 'maven-3'
    }
    environment {

    // App info
    APP_NAME = "register-app-pipeline"

    // Docker Hub username
    DOCKER_USER = "hndevghazali"

    // Image ka base naam
    IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"

    // 🔒 FIXED TAG (kabhi change nahi hota)
    // Har build mein naya banega
    FIXED_TAG = "1.0.0-${BUILD_NUMBER}"

    // 🔄 ROLLING TAG (hamesha latest)
    ROLLING_TAG = "latest"
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
        stage("Build & Push Docker Image") {
    steps {
        script {
            docker.withRegistry('https://registry.hub.docker.com', 'hn') {

                def image = docker.build(
                    "${IMAGE_NAME}:${FIXED_TAG}"
                )

                // 🔒 Fixed version push
                image.push()

                // 🔄 Rolling tag push
                image.push("${ROLLING_TAG}")
            }
        }
    }
}
        stage("Update GitOps Repo") {
  steps {
    withCredentials([usernamePassword(
      credentialsId: 'github',
      usernameVariable: 'GIT_USER',
      passwordVariable: 'GIT_TOKEN'
    )]) {
      sh """
        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/devopsghazali/ci_cd_automation.git
        cd ci_cd_automation/k8s
        sed -i 's|image: .*|image: hndevghazali/register-app:${FIXED_TAG}|' deployment.yaml
        git add deployment.yaml
        git commit -m "Update image to ${FIXED_TAG}"
        git push
      """
    }
  }
}


    }
}

pipeline {
    agent any
    tools {
        jdk 'Jdk17'
        gradle 'gradle'
    }
    stages {
        stage('checkout from scm') {
            steps {
                git branch: 'main', url: 'https://github.com/mariyam-q/Java_Gradle_Responsive_Website.git'
            }
        }
        stage('Gradle compile') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew compileJava'
            }
        }
        stage('Test Gradle') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew test'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }
                    // Quality gate
                    timeout(time: 10, unit: 'MINUTES')
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline is aborted due to quality gate failure: ${qg.status}"
                    }
                }
            }
        }
        stage('Build Gradle') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew build'
            }
        }
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'dp-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.html'
            }
        }
        stage('Build and Push to Nexus') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh '''
                        docker build -t 20.70.136.109:8083/gradle1:latest .
                        docker login -u admin -p $docker_password 20.70.136.109:8083
                        docker push 20.70.136.109:8083/gradle1:latest
                        '''
                    }
                }
            }
        }
        stage('Deploy to Container') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh 'docker run -d --name g1 -p 8082:8080 20.70.136.109:8083/gradle1:latest'
                    }
                }
            }
        }
    }
}

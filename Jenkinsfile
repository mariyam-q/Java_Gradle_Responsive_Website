pipeline {
agent any
tools {
jdk 'Jdk17'
gradle 'gradle'
}
stages {
stage('Checkout from SCM') {
steps {
git branch: 'main', url: 'https://github.com/mariyam-q/Java_Gradle_Responsive_Website.git'
}
}
stage('Gradle Compile') {
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
stage('SonarQube Analysis') {
steps {
script {
withSonarQubeEnv(credentialsId: 'sonar_token') {
sh 'chmod +x gradlew'
sh './gradlew sonarqube'
}
// Quality gate
timeout(time: 10, unit: 'MINUTES') {
def qg = waitForQualityGate()
if (qg.status != 'OK') {
error "Pipeline is aborted due to quality gate failure: ${qg.status}"
}
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
dependencyCheck additionalArguments: '--scan ./ --format HTML', odcInstallation: 'dp-check'
dependencyCheckPublisher pattern: '**/dependency-check-report.html'
}
}
stage('Build Docker Image') {
steps {
script {
withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'mariyamq', passwordVariable: 'mariyamV@123')]) {
                        sh '''
                        sudo docker build -t mariyamq/gradle1:latest .
                       sudo  docker login -u mariyamq -p mariyamV@123
                        '''
}
}
}
}
stage('Docker Image Scan') {
steps {
sh "sudo trivy image --format table -o trivy-image-report.html mariyamq/gradle1:latest"
}
}
stage('Push Docker Image to Nexus') {
            steps {
                script {
                    //withCredentials([usernamePassword(credentialsId: 'docker_pass', usernameVariable: 'admin', passwordVariable: 'mariyamV@123')]) {
                        sh '''
                        sudo docker tag mariyamq/gradle1:latest 20.70.136.109:8083/gradle1:latest
                        sudo docker login -u admin -p mariyamV@123 20.70.136.109:8083
                        sudo docker push 20.70.136.109:8083/gradle1:latest
                        '''
                    //}
                }
            }
        }
stage('Deploy to Container') {
steps {
script {
sh 'sudo docker run -d --name g1 -p 8082:8080 20.70.136.109:8083/gradle1:latest'
}
}
}
}
post {
always {
cleanWs()
}
}
}

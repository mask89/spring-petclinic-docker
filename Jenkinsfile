pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockertest')
    }
    stages {
        stage('Build') {
            steps {
                sh '/usr/local/maven/bin/mvn -B -DskipTests clean package'
            }
        }
        stage('Test') { 
            steps {
                sh '/usr/local/maven/bin/mvn test' 
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml' 
                }
            }
        }
        stage('OWASP Check') {
            steps {
                dependencyCheck additionalArguments: '--format HTML', odcInstallation: 'DP-Check'
            }
        }
        stage("Build & SonarQube analysis") {
            agent any
            steps {
              withSonarQubeEnv('sq1') {
                sh "/usr/local/maven/bin/mvn -Dcheckstyle.skip clean package sonar:sonar -D sonar.projectKey=Demo -D sonar.projectName='Demo' -D sonar.host.url=http://172.18.25.204:9000/ -D sonar.token=sqp_014bd86730736b7de34fd2bf877c0aef956bc53e"
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
        stage('Docker Build'){
            steps{
                sh 'docker build . -t sunguyen88/petclinic:0.1'
            }
        }
        stage('Docker Push'){
            steps{
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push sunguyen88/petclinic:0.1'
            }
            
        }
        stage('Trivy'){
            steps{
                sh 'trivy image --scanners vuln petclinic:0.1'
            }
        }
    }
    post{
        always{
            sh 'docker logout'
        }
    }
}
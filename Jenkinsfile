pipeline {
 agent { node { label "sonar-maven" } }
 
 environment {
        AWS_REGION = 'eu-west-1'
        ECR_REPO = 'addressbook_teamb'
        AWS_ACCOUNT_ID = '${params.aws_account}'
        URL_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
 }
 parameters   {
   choice(name: 'aws_account',choices: ['249641587437', '4568366404742', '922266408974'], description: 'aws account hosting image registry')
   choice(name: 'ecr_tag',choices: ['1.1.0','1.2.0','1.3.0'], description: 'Choose the ecr tag version for the build')
       }
tools {
    maven "Maven"
    }
    stages {
      stage('1. Git Checkout') {
        steps {
          git branch: 'main', credentialsId: 'github-repo-pat', url: 'https://github.com/TeamB-34/addressbook2.git'
        }
      }
      stage('2. Build with maven') { 
        steps{
          sh "mvn clean package"
         }
       }
      stage('3. SonarQube analysis') {
      environment {SONAR_TOKEN = credentials('sonar-token-abook')}
      steps {
       script {
         def scannerHome = tool 'SonarQube_Scanner-5.0.1';
         withSonarQubeEnv("sonar-integration") {
         sh "${tool("SonarQube_Scanner-5.0.1")}/bin/sonar-scanner -X \
             -Dsonar.projectKey=Addressbook \
             -Dsonar.projectName='Addressbook' \
             -Dsonar.host.url=http://18.201.23.183:9000 \
             -Dsonar.token=$SONAR_TOKEN \
             -Dsonar.sources=src/main/java/ \
             -Dsonar.java.binaries=target/classes"
          }
         }
       }
      }
      stage ('Docker image build and push to ECR') { 
        steps {
            script {
           withCredentials([usernamePassword(credentialsId: 'ecr-demo-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]){
          sh "aws ecr get-login-password --region eu-west-1 | sudo docker login --username AWS --password-stdin ${params.aws_account}.dkr.ecr.eu-west-1.amazonaws.com"
          sh "sudo docker build -t $ECR_REPO ."
          sh "sudo docker tag addressbook_teamb:latest ${params.aws_account}.dkr.ecr.eu-west-1.amazonaws.com/addressbook_teamb:${params.ecr_tag}"
          sh "sudo docker push ${params.aws_account}.dkr.ecr.eu-west-1.amazonaws.com/addressbook_teamb:${params.ecr_tag}"
         }
        }
       }
      }
      stage('5. Deployment into kubernetes cluster') {
        steps{
          kubeconfig(caCertificate: '',credentialsId: 'k8s-kubeconfig', serverUrl: '') {
          sh "/snap/bin/kubectl apply -f manifest"
          }
         }
       }

      stage ('6. Email Notification') {
         steps{
         mail bcc: 'opeyemidavies@gmail.com', body: '''Build is Over. Check the application using the URL below. 
         http://54.171.49.113:8080/
         Let me know if the changes look okay.
         Thanks,
         Dominion System Technologies,
         +1 (313) 413-1477''', cc: 'akendokobe@gmail.com', from: '', replyTo: '', subject: 'Application was Successfully Deployed!!', to: 'akendokobe@gmail.com'
      }
    }
 }
}



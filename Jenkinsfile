pipeline {
    agent any
    
    environment {
        SonarQubeTool = tool name: 'sonar_scanner_dotnet'
        Docker_Repository = 'app_prateeksharma01'
        Docker_Login_User = credentials('DockerLoginUser')
        Docker_Login_Password = credentials('DockerLoginPassword')
        UserName = 'prateeksharma01'
    }
    
    stages {
        stage('Clean') {
            steps {
                cleanWs()
            }
        }
       
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/prateeksharma01/app_prateeksharma01.git'
            }
        }
        stage('Nuget restore') {
            steps {
                bat "dotnet restore nagp-devops-us/nagp-devops-us.csproj"
            }
        }
//        stage('Sonarqube Begin') {
//            steps {
//                withSonarQubeEnv('Test_Sonar') {
//                    bat "${SonarQubeTool}\\SonarScanner.MSBuild.exe begin /k:sonar-${UserName} /n:sonar-${UserName} /o:sonar-prateeksharma01 /v:1.0 /d:sonar.cs.vstest.reportsPaths=**/*.trx /d:sonar.cs.vscoveragexml.reportsPaths=**/*.coverage"
//                }
//            }
//        }
        stage('Build') {
            steps {
                bat "dotnet build"
            }
        }
//        stage('Test') {
//            steps {
//                bat 'dotnet test --logger "trx;LogFileName=nagp-devops-us.Tests.Results.trx" --no-build --collect "Code Coverage"'
//                mstest testResultsFile:"**/*.trx", keepLongStdio: true
//            }
//        }
//        stage('Sonarqube End') {
//            steps {
//                withSonarQubeEnv('Test_Sonar') {
//                    bat "${SonarQubeTool}\\SonarScanner.MSBuild.exe end"
//                }
//            }
//        }
//        stage('Publish') {
//            steps {
//                bat 'dotnet publish nagp-devops-us -o Publish -c Release'
//                bat 'docker rmi -f nagp-devops-us:local_dev'
//                bat "docker build -f ${WORKSPACE}\\Publish\\Dockerfile -t nagp-devops-us:local_dev ${WORKSPACE}\\Publish"
//                bat "docker tag nagp-devops-us:local_dev ${Docker_Login_User}/i-${UserName}-${env.BRANCH_NAME}:latest"
//                bat "docker login -u ${Docker_Login_User} -p ${Docker_Login_Password}"
//                bat "docker push ${Docker_Login_User}/i-${UserName}-${env.BRANCH_NAME}:latest"
//            }
//        }
        
        stage('Deploy') {
            steps {
                bat "gcloud container clusters get-credentials nagp-devops --zone us-central1-c --project liquid-receiver-357413"
                bat "kubectl apply -f deploymentandservice.yaml"
            }
        }
    }
}
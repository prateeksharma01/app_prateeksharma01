pipeline {
    agent any
    
    environment {
        SonarQubeTool = tool name: 'sonar_scanner_dotnet'
        //Docker_Repository = 'app_prateeksharma01'
        //Docker_Login_User = credentials('DockerLoginUser')
        //Docker_Login_Password = credentials('DockerLoginPassword')
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
        stage('Sonarqube Begin') {
            steps {
                withSonarQubeEnv('Test_Sonar') {
                    bat "${SonarQubeTool}\\SonarScanner.MSBuild.exe begin /k:sonar-${UserName} /n:sonar-${UserName} /v:1.0 /d:sonar.cs.vstest.reportsPaths=**/*.trx /d:sonar.cs.vscoveragexml.reportsPaths=**/*.coverage"
                }
            }
        }
        stage('Build') {
            steps {
                bat "dotnet restore nagp-devops-us/nagp-devops-us.csproj"
                bat "dotnet build"
            }
        }
        stage('Test') {
            steps {
                bat 'dotnet test --logger "trx;LogFileName=nagp-devops-us.Tests.Results.trx" --no-build --collect "Code Coverage"'
                mstest testResultsFile:"**/*.trx", keepLongStdio: true
            }
        }
        stage('Sonarqube End') {
            steps {
                withSonarQubeEnv('Test_Sonar') {
                    bat "${SonarQubeTool}\\SonarScanner.MSBuild.exe end"
                }
            }
        }
        stage('Publish') {
            steps {
                bat 'dotnet publish DevOpsnMicroServices -o Publish -c Release'
                bat 'docker rmi -f devopsnmicroservices:local_dev'
                bat "docker build -f ${WORKSPACE}\\Publish\\Dockerfile -t devopsnmicroservices:local_dev ${WORKSPACE}\\Publish"
                bat "docker tag devopsnmicroservices:local_dev ${Docker_Login_User}/i-${UserName}-master:dev_${BUILD_NUMBER}"
                bat "docker login -u ${Docker_Login_User} -p ${Docker_Login_Password}"
                bat "docker push ${Docker_Login_User}/i-${UserName}-master:dev_${BUILD_NUMBER}"
            }
        }
        stage('Deploy') {
            steps {
                bat "docker rm c-${UserName}_master -f"
                bat "docker run -p 7100:7100 -d --name c-${UserName}_master ${Docker_Login_User}/i-${UserName}-master:dev_${BUILD_NUMBER}"
            }
        }
    }
}
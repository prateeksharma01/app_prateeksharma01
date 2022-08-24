pipeline {
    agent {
                label 'ubuntu'
            } 
    
    environment {
        SonarQubeTool = tool name: 'sonar_scanner_dotnet'
        Docker_Repository = 'app_prateeksharma01'
        Docker_Login_User = credentials('DockerLoginUser')
        Docker_Login_Password = credentials('DockerLoginPassword')
        UserName = 'prateeksharma01'
    }
    
    stages {
       
        stage('Nuget restore') {
            steps {
                cleanWs()
                git branch: "${GIT_BRANCH}", url: 'https://github.com/prateeksharma01/app_prateeksharma01.git'
                sh "dotnet restore nagp-devops-us/nagp-devops-us.csproj"

            }
        }
        stage('Start sonarqube analysis') {
              agent { 
                label 'windows'
              }
            when {
                     branch 'master'
                }
            steps {
                
                withSonarQubeEnv('Test_Sonar') {
                     bat "C:\ProgramData\Jenkins\.jenkins\tools\hudson.plugins.sonar.MsBuildSQRunnerInstallation\sonar_scanner_dotnet\SonarScanner.MSBuild.exe begin /k:sonar-${UserName} /n:sonar-${UserName} /o:sonar-prateeksharma01 /v:1.0 /d:sonar.cs.vstest.reportsPaths=**/*.trx /d:sonar.cs.vscoveragexml.reportsPaths=**/*.coverage"
                }
            }
        }
        stage('Code build') {
            agent { 
                label 'windows'
              }
            steps {
                 bat "dotnet restore nagp-devops-us/nagp-devops-us.csproj"
                 bat "dotnet build"
            }
        }
        stage('Test case execution') {
            agent { 
                label 'windows'
              }
            when {
                     branch 'master'
                }
            steps {
                 bat 'dotnet test --logger "trx;LogFileName=nagp-devops-us.Tests.Results.trx" --no-build --collect "Code Coverage"'
                mstest testResultsFile:"**/*.trx", keepLongStdio: true
            }
        }
        stage('Release artifact') {
            steps {
                 sh 'dotnet publish nagp-devops-us -o Publish -c Release'
            }
        }
        
        stage('Stop sonarqube analysis') {
              agent { 
                label 'windows'
              }
            when {
                     branch 'master'
                }
            steps {
                withSonarQubeEnv('Test_Sonar') {
                     bat "${SonarQubeTool}\\SonarScanner.MSBuild.exe end"
                }
            }
        }
        stage('Docker publish') { 
            

            steps {
                 sh 'sudo docker rmi -f nagp-devops-us:local_dev'
                 sh "sudo docker build -f ${WORKSPACE}/Publish/Dockerfile -t nagp-devops-us:local_dev ${WORKSPACE}/Publish"
                 sh "sudo docker tag nagp-devops-us:local_dev ${Docker_Login_User}/i-${UserName}-${env.BRANCH_NAME}:latest"
                 sh "sudo docker login -u ${Docker_Login_User} -p ${Docker_Login_Password}"
                 sh "sudo docker push ${Docker_Login_User}/i-${UserName}-${env.BRANCH_NAME}:latest"
            }
        }
        stage('Kubernetes Deployment') {
            steps {
                 sh "gcloud container clusters get-credentials nagp-devops --zone us-central1-c --project liquid-receiver-357413"
                 sh "sed -i \"s/#BRANCH#/\${GIT_BRANCH}/\" ${WORKSPACE}/deploymentandservice.yaml"
                 sh "kubectl apply -f deploymentandservice.yaml"
            }
        }
    }
}
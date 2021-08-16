pipeline{
    agent any
    environment{
        username= "bhavinibatra"
        registry= "bhavinibatra/biannual"
        scannerHome= tool "sonar_scanner_dotnet"
    }
    stages{
        stage('Checkout from GitHub'){
            steps{
                checkout([   $class: 'GitSCM',
         branches: [[name: '*/feature']],
         userRemoteConfigs: [[credentialsId: 'GitCreds', url: 'https://github.com/BhaviniB/DevOpsBiAnnual']]
       ])
            }
        }
        stage('Sonar start'){
            withSonarQubeEnv('Test_Sonar'){
                bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:biannualkey /n:biannualname /v:1.0"
            }
        }
        stage('Build code for feature'){
            steps{
                bat "dotnet build WebApplication4\\WebApplication4.csproj -c Release -o WebApplication4/app/build"
            }
        }
        stage('Execute test cases'){
            steps{
                bat "dotnet test TestProject2"
            }
        }
        stage('Stop sonar'){
            steps{
             withSonarQubeEnv('Test_Sonar'){
                bat "${scannerHome}\\SonarScanner.MSBuild.exe end"
            }
            }
        }
        stage('Build docker image'){
            steps{
            bat "docker build -t i-biannual-feature --no-cache ."
            }
        }
        stage('Push to dockerhub'){
            steps{
                bat "docker tag i-biannual-feature ${registry}:${BUILD_NUMBER}"
                bat "docker tag i-biannual-feature ${registry}:latest"
                withDockerRegistry([credentialsId: 'DockerHub', url:'']){
                    bat "docker push ${registry}:${BUILD_NUMBER}"
                    bat "docker push ${registry}:latest"
                }
            }
        }
        stage('Pre container check'){
            steps{
                environment{
                    CONTAINER_ID = "${bat(script:'docker ps -aqf name="^c-biannual$"', returnStdout: true).trim().readLines().drop(1).join("")}"
                }
                script{
                    if(env.CONTAINER_ID!=null){
                        bat "docker rm -f c-biannual"
                    }
                    else{
                        echo "container removal not needed"
                    }
                }

            }
            
        }
        stage('Deployment'){
            parallel{
                stage('Doker dep'){
                    steps{
                        bat "docker run --name c-biannual -d -p 7400:80 ${registry}:${BUILD_NUMBER}"
                    }
                    

                }
                stage('Kubernetes deployment'){
                    steps{
                        bat "kubectl apply -f deployment.yaml"
                    }
                }
            }
        }
    }
}
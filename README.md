## .net files for Jenkins CI/CD Course

Install .net SDK in Ubuntu22:
```
sudo apt install dotnet-sdk-6.0
dotnet --version
```
create a sample app:
```
dotnet new mvc --name dotnetwebapp
```
Run the application:
```
cd dotnetwebapp
dotnet run /or dotnet run --urls http://*:5000
```

## Jenkins Pipeline

pipeline {
    agent any

    environment {
        registry = "kss7/dothecks"
        img = "${registry}:${env.BUILD_ID}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Search-Pankaj/dotnetapp.git'
            }
        }
stage('Stop running Container') {
    steps {
        script {
            sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${JOB_NAME} | awk "{print \$3}")' 
            sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk "{print \$3}") --force'
            sh returnStatus: true, script: 'docker rm ${JOB_NAME}'
           
        }
    }
}        
        stage('Build') {
            steps {
                echo "Building our image"
                script {
                    docker.build("${img}")
                }
            }
        }
        stage('Run') {
            steps {
                echo "Run image"
                script {
                    sh returnStdout: true, script: "docker run --rm -d --name ${JOB_NAME} -p 8081:5000 ${img}"
                }
            }
        }
    }
}


pipeline 
{
    agent any
    
    tools
    {
        maven 'maven3'
        nodejs 'node18'
    }
    
    environment
    {
        SCANNER_HOME = tool 'sonar-scanner'
        
        workspacecleanup=true
        gitcheckout=true
        mvncompile=true
        mvntest=true
        trivyfsscan=true
        sonarqubeanalysis=true
        mvnbuild=true
        imagebuild=true
        trivyimagescan=true
        imagepushtodockerhub=true
        deploymenttominikube=true
    }

    stages 
    {
        stage('Workspace Cleanup') 
        {
            when { expression { workspacecleanup == 'true'}}
            steps 
            {
                cleanWs()
            }
        }
        
        stage('Git-Checkout')
        {
            when { expression { gitcheckout == 'true' } }
            steps
            {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/ganesh-legend/Multi-Tier-With-Database.git'
            }
        }
        
        stage('Maven compile')
        {
            when { expression { mvncompile == 'true' } }
            steps
            {
                sh "mvn clean compile"
            }
        }
        
        stage('Maven Test')
        {
            when { expression { mvntest == 'true' } }
            steps
            {
                sh "mvn clean test"
            }
        }
        
        stage('Trivy FS Scan')
        {
            when { expression { trivyfsscan == 'true' } }
            steps
            {
                sh "trivy fs ."
            }
        }
        
        stage('Sonarqube Analysis')
        {
            when { expression { sonarqubeanalysis == 'true' } }
            steps
            {
                withSonarQubeEnv('sonar-server') 
                {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner \
                       -Dsonar.projectName=GoldenCatBankApp \
                       -Dsonar.projectKey=GoldenCatBankApp \
                       -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('Maven Build')
        {
            when { expression { mvnbuild == 'true' } }
            steps
            {
                sh "mvn clean package"
            }
        }
        
        stage('Image build')
        {
            when { expression { imagebuild == 'true' } }
            steps
            {
                script
                {
                    withDockerRegistry(credentialsId: 'dockerhub-token', toolName: 'docker') 
                    {
                        sh "docker build . -t ganesh1326/goldencatbankapp:latest"
                    }
                }
            }
        }
        
        stage('Trivy Image Scan')
        {
            when { expression { trivyimagescan == 'true' } }
            steps
            {
                sh "trivy image ganesh1326/goldencatbankapp:latest"
            }
        }
        
        stage('Image Push to Dockerhub')
        {
            when { expression { imagepushtodockerhub == 'true' } }
            steps
            {
                script
                {
                    withDockerRegistry(credentialsId: 'dockerhub-token', toolName: 'docker') 
                    {
                        sh "docker push ganesh1326/goldencatbankapp:latest"
                    }
                }
            }
        }
        
        stage('Deployment To Minikube')
        {
            when { expression { deploymenttominikube == 'true' } }
            steps
            {
                script
                {
                    kubeconfig(credentialsId: 'Minikube-K8s-Cloud-goldencatbankapp-deployment-token', serverUrl: 'https://192.168.49.2:8443') 
                    {
                         sh "kubectl apply -f ds.yml"
                         sh "kubectl apply -f php-admin-for-mysql-deploy-and-service.yaml"
                         sh "kubectl port-forward svc/bankapp-service -n goldencatbankapp 1000:80 --address 0.0.0.0 &"
                         sh "kubectl port-forward svc/phpmyadmin -n goldencatbankapp 2000:80 --address 0.0.0.0 &"
                    }
                }
            }
        }
    }
    
    post 
    {
        success 
        {
          emailext (
              attachLog: true,
              subject: "Build Successful: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
              body: "Good news! The build was successful.\n\nJob: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\nBuild URL: ${env.BUILD_URL}",
              recipientProviders: [[$class: 'DevelopersRecipientProvider']],
              to: 'gp420083@gmail.com'
          )
        }
          
        failure 
        {
          emailext (
              attachLog: true,
              subject: "Build Failed: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
              body: "Unfortunately, the build failed.\n\nJob: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\nBuild URL: ${env.BUILD_URL}",
              recipientProviders: [[$class: 'DevelopersRecipientProvider']],
              to: 'gp420083@gmail.com'
          )
        }
    }
}

try{
	node('master'){
            def docker
            def dockerCMD
            def jdk
            def java
            def dockerImage
            def tagName
            def gitURL
            def projectName
            stage('Prepare'){
                echo "Prepare Tools and Variables to be used in Pipeline Script"
                docker = tool name: 'docker', type: 'dockerTool'
                dockerCMD = "${docker}/bin/docker"
                jdk = tool name: 'java11', type: 'jdk'
                java = "${jdk}/bin/java"
                dockerImage = "mdanwarjamal/edureka-devops-cert-app"
                tagName = "1.0"
                gitURL = "https://github.com/mdanwarjamal"
                projectName = "edureka-devops-cert-project"
            }
            stage('Checkout'){
                echo "Checkout Application Code from GitHub Repository"
                git branch: 'main', credentialsId: 'GitHub', url: "${gitURL}/${projectName}.git"
	        }
	        stage('Provision Tool on Test Server'){
	            echo "Provisioning Tools required for deployment and testing PHP Application Deployment on Test Servers"
	            ansiblePlaybook credentialsId: 'AnsibleSSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'testServerTools.yml'
	        }
            stage('Build Docker Image'){
                echo "Build Docker image from Dockerfile"
                sh "sudo ${dockerCMD} build -t ${dockerImage}:${tagName} ."
            }
            stage('Push to DockerHub'){
                echo "Push Docker Image to DockerHub Online Registry"
                withCredentials([usernamePassword(credentialsId: 'DockerHub', passwordVariable: 'dockerHubPwd', usernameVariable: 'dockerHubUsername')]) {
                    sh "sudo ${dockerCMD} login -u ${dockerHubUsername} -p ${dockerHubPwd}"
                    sh "sudo ${dockerCMD} push  ${dockerImage}:${tagName}"
                }
            }
            stage('Deploy Application on Test Server'){
                echo "Deploying containerized PHP Application on Test Servers"
                ansiblePlaybook credentialsId: 'AnsibleSSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'testServerDeployment.yml'
            }
            stage('Automation Test on Test Server'){
                echo "Testing the Application Deployment of PHP Application on Test Servers"
		try{
		    ansiblePlaybook credentialsId: 'AnsibleSSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'seleniumTestDeploy.yml'
		}catch(Exception e){
		    echo "Stopping and Removing all COntainers on Test Server"
		    ansiblePlaybook credentialsId: 'AnsibleSSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'StopAppOnTestServer.yml'
		    throw e;
		}
            }
	    stage('Provision Tool on Prod Server'){
	        echo "Provisioning Tools required for deployment PHP Application Deployment on Prod Servers"
	        ansiblePlaybook credentialsId: 'AnsibleSSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'prodServerTools.yml'
	    }
            stage('Deploy Application on Prod Server'){
                echo "Deploying containerized PHP Application on Prod Servers"
                ansiblePlaybook credentialsId: 'AnsibleSSH', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'prodServerDeployment.yml'
            }
            currentBuild.result = 'SUCCESS'
        }
}
catch(Exception ex){
        echo "Some Exception Occured"
        currentBuild.result = 'FAILURE'
}

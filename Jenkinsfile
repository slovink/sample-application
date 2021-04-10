pipeline {
    agent {
    node {
        label 'blue-green'
        customWorkspace '/home/jenkins/workspace/'
    }
}
    parameters {
        choice(
            choices: ['Create', 'Test' , 'Build' , 'Deploy', 'Delete', 'Rollback'],
            description: '',
            name: 'REQUESTED_ACTION')
        choice(
            choices: ['slv'],
            description: '',
            name: 'ProjectName')
        choice(
            choices: ['DEV', 'UAT', 'PROD'],
            description: '',
            name: 'EnvironmentName')
		choice(
		    choices: ['sample', 'app1' , 'app2' , 'app3'],
			description: 'Select Application Which you want to deploy',
			name: 'ServiceName')
		string(defaultValue: "master", description: 'What BranchName?', name: 'BranchName')    
    string(defaultValue: "latest", description: 'What buildVersion?', name: 'BuildVersion')
		string(defaultValue: "https://github.com/slovink/" , description: 'Source Code', name: 'SourceCodeRepo')
		string(defaultValue: "kubecode", description: 'Docker Repository ', name: 'DockerRegistry')
    }
    stages {
        
        
        stage ('Prepare') {
            steps {
                echo "preparing...."
                cleanWs()
                 checkout([$class: 'GitSCM',
                     branches: [[name: "origin/${BranchName}"]],
                     doGenerateSubmoduleConfigurations: false,
                     extensions: [[$class: 'LocalBranch']],
                     submoduleCfg: [],
                userRemoteConfigs: [[
                         url: '${SourceCodeRepo}/${ServiceName}-application.git']]])     
            }
        }
        stage('Build') {
            when {
                
                expression { params.REQUESTED_ACTION == 'Build' || params.REQUESTED_ACTION == 'Test' }
            }
            steps {
                echo 'Building..'
				
                sh 'echo Testing' 
                            }
        }
        stage('Docker build and publish') {
            when {
                expression { params.REQUESTED_ACTION == 'Build'  }
            }
            steps {
                
	        sh 'docker build -t ${ServiceName}:${BuildVersion} .'
                sh 'docker tag ${ServiceName}:${BuildVersion}  ${DockerRegistry}/${ServiceName}:${BuildVersion}-${EnvironmentName}-${BranchName}'
                sh 'docker login'
                sh 'docker push ${DockerRegistry}/${ServiceName}:${BuildVersion}-${EnvironmentName}-${BranchName}'
                
            }
        }
    
        stage('Deploy') {
		    when {
			    expression { params.REQUESTED_ACTION == 'Deploy'  }
              
            steps {
                echo 'Deploying....'
			    
                                        
                }
				
               }
		stage('Delete') {
		    when {
			    expression { params.REQUESTED_ACTION == 'Delete'  }
              
            steps {
                echo 'Destroying....'
		   
                                        
                }
				
               }	   
        }
    }

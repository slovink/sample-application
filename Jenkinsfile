pipeline {
    agent {
    node {
        label 'blue-green'
        customWorkspace '/home/jenkins/workspace/'
    }
}
    parameters {
        choice(
            choices: ['Build' , 'AddService' , 'Deploy', 'Delete', 'Rollout'],
            description: '',
            name: 'REQUESTED_ACTION')
        choice(
            choices: ['dev','uat','prod'],
            description: '',
            name: 'NameSpace')
        choice(
            choices: ['DEV', 'UAT', 'PROD'],
            description: '',
            name: 'EnvironmentName')
        string(defaultValue: "main", description: 'What BranchName?', name: 'BranchName')    
        string(defaultValue: "sample", description: 'What servicename?', name: 'ServiceName')
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
                
                expression { params.REQUESTED_ACTION == 'Build' || params.REQUESTED_ACTION == 'AddService' }
            }
            steps {
                echo 'Building..'
                
                sh 'docker build -t ${ServiceName}:${BuildVersion} .'

            }
        }
        stage('Docker build and publish') {
            when {
                expression { params.REQUESTED_ACTION == 'Build' || params.REQUESTED_ACTION == 'AddService' }
            }
            steps {
                
                sh 'docker tag ${ServiceName}:${BuildVersion} ${DockerRegistry}/${ServiceName}:${EnvironmentName}-${BranchName}-${BUILD_NUMBER}'
                sh 'docker push ${DockerRegistry}/${ServiceName}:${EnvironmentName}-${BranchName}-${BUILD_NUMBER}'
                
            }
        }
    
        stage('Deploy') {
              
            steps {
                echo 'Deploying....'
	     }
		    
             steps {
                echo "preparing Helm Repo...."
                cleanWs()
                 checkout([$class: 'GitSCM',
                     branches: [[name: "origin/${BranchName}"]],
                     doGenerateSubmoduleConfigurations: false,
                     extensions: [[$class: 'LocalBranch']],
                     submoduleCfg: [],
                     userRemoteConfigs: [[
                         url: 'https://github.com/slovink/argocd-bluegreen.git']]]) 
	     }
		
        }
    }
}

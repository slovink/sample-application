pipeline {
    agent {
    node {
        label 'blue-green'
        customWorkspace '/home/jenkins/workspace/'
    }
}
    parameters {
        choice(
            choices: ['Create' , 'Update', 'Delete', 'Rollout'],
            description: '',
            name: 'REQUESTED_ACTION')
        choice(
            choices: ['dev','uat','prod'],
            description: '',
            name: 'NameSpace')
        choice(
            choices: ['dev', 'uat', 'prod'],
            description: '',
            name: 'EnvironmentName')
        string(defaultValue: "main", description: 'What BranchName?', name: 'BranchName')    
        string(defaultValue: "sample", description: 'What servicename?', name: 'ServiceName')
        string(defaultValue: "latest", description: 'What buildVersion? Required when you are Choosing Rollout Option', name: 'BuildVersion')
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
                
                expression { params.REQUESTED_ACTION == 'Build' || params.REQUESTED_ACTION == 'Create' }
            }
            steps {
                echo 'Building..'
                
                sh 'docker build -t ${ServiceName}:${BuildVersion} .'

            }
        }
        stage('Docker build and publish') {
            when {
                expression { params.REQUESTED_ACTION == 'Build' || params.REQUESTED_ACTION == 'Create' }
            }
            steps {
                
                sh 'docker tag ${ServiceName}:${BuildVersion} ${DockerRegistry}/${ServiceName}:${EnvironmentName}-${BranchName}-${BUILD_NUMBER}'
                sh 'docker push ${DockerRegistry}/${ServiceName}:${EnvironmentName}-${BranchName}-${BUILD_NUMBER}'
                
            }
        }

        stage ('Prepare Helm Repository') {
            steps {
                echo "preparing...."
                cleanWs()
                 checkout([$class: 'GitSCM',
                     branches: [[name: "origin/master"]],
                     doGenerateSubmoduleConfigurations: false,
                     extensions: [[$class: 'LocalBranch']],
                     submoduleCfg: [],
                     userRemoteConfigs: [[
                         url: 'https://github.com/slovink/argocd-bluegreen.git']]]) 
            }
        }
	    
       stage('Deploy') {
	               
            steps {
                echo 'Deploying....'
                
                script {
                   
                    sh 'pwd;'
                    
                    
                if ("${REQUESTED_ACTION}"=='Create')
                
                { sh'ls;'
                sh'helm install  ${ServiceName}-${EnvironmentName} ${ServiceName} --set name=${ServiceName},namespace=${NameSpace},image.tag=${EnvironmentName}-${BranchName}-${BUILD_NUMBER},image.repository=${DockerRegistry}/${ServiceName} --debug -f ${ServiceName}/values.yaml --namespace ${NameSpace};'}
                else
                {sh 'helm upgrade ${ServiceName}-${EnvironmentName} ${ServiceName} --set name=${ServiceName},namespace=${NameSpace},image.tag=${EnvironmentName}-${BranchName}-${BUILD_NUMBER},image.repository=${DockerRegistry}/${ServiceName} --namespace ${NameSpace} --debug -f ${ServiceName}/values.yaml '}              
                
                }
               }
        }
       stage('Rollout') {
	               
            steps {
                echo 'Deploying....'
                
                script {
                   
                    sh 'pwd;'
                    
                    
                if ("${REQUESTED_ACTION}"=='Rollout')
                
                { sh'ls;'
                sh 'helm upgrade ${ServiceName}-${EnvironmentName} ${ServiceName} --set name=${ServiceName},namespace=${NameSpace},image.tag=${BuildVersion},image.repository=${DockerRegistry}/${ServiceName} --namespace ${NameSpace} --debug -f ${ServiceName}/values.yaml '}
            
                }
               }
        }	    
	    
    }
}

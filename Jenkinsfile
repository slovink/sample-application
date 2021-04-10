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
        string(defaultValue: "dev", description: 'What BranchName?', name: 'BranchName')    
        string(defaultValue: "sample", description: 'What servicename?', name: 'ServiceName')
        string(defaultValue: "latest", description: 'What buildVersion?', name: 'BuildVersion')
        string(defaultValue: "5004", description: 'ServicePort', name: 'ServicePort')
        string(defaultValue: "ClusterIP", description: 'servicetype', name: 'ServiceType')
	string(defaultValue: "https://github.com/slovink/" , description: 'Source Code', name: 'SourceCodeRepo')
        


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
                
                sh 'docker tag ${ServiceName}:${BuildVersion} ${DockerRegistry}/${ServiceName}:${BuildVersion}-${EnvironmentName}-${BranchName}'
                sh 'docker push ${DockerRegistry}/${ServiceName}:${BuildVersion}-${EnvironmentName}-${BranchName}'
                
            }
        }
    
        stage('Deploy') {
              
            steps {
                echo 'Deploying....'
                
                script {
                    sh 'helm init;'
                    sh 'pwd;'
                    
                    
                if ("${REQUESTED_ACTION}"=='AddService')
                
                { sh'ls;'
                sh'helm install --name ${ServiceName}-${EnvironmentName} Charts --set name=${ServiceName},namespace=${NameSpace},image.tag=${BuildVersion}-${EnvironmentName}-${BranchName},service.type=${ServiceType},service.port=${ServicePort},image.repository=${DockerRegistry}/${ServiceName},resources.limits.cpu=${CpuLimit},resources.requests.cpu=${CpuRequests},resources.limits.memory=${MemoryLimit},resources.requests.memory=${MemoryRequests},consulAddress=${ConsulAddress},replicaCount=${ReplicaCount} --debug -f Charts/values.yaml --namespace ${NameSpace};'}
                else
                {sh 'helm upgrade ${ServiceName}-${EnvironmentName} Charts --set name=${ServiceName},namespace=${NameSpace},image.tag=${BuildVersion}-${EnvironmentName}-${BranchName},service.type=${ServiceType},service.port=${ServicePort},image.repository=${DockerRegistry}/${ServiceName},resources.limits.cpu=${CpuLimit},resources.requests.cpu=${CpuRequests},resources.limits.memory=${MemoryLimit},resources.requests.memory=${MemoryRequests},consulAddress=${ConsulAddress},replicaCount=${ReplicaCount} --namespace ${NameSpace} --debug -f Charts/values.yaml --recreate-pods'}
                
                
                }
               }
        }
    }
}

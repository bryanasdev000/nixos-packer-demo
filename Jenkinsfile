pipeline {
  agent {
    label 'master'
  }
   parameters {
        choice(name: "type", choices: ["web","db"], description: "Tipo de maquina a ser construida a imagem")
    }
  stages {
    stage("Preparation - packer") {
      steps {
        sh("packer version")
        sh("echo Bulding ${params.type} type config")
      }
    }
    stage("Validate packer template") {
      steps {
        withCredentials([string(credentialsId: 'project_id', variable: 'GCP_PROJECT_ID'), file(credentialsId: 'gcp_service_account_file', variable: 'GCP_SERVICE_ACCOUNT_FILE'), file(credentialsId: 'ssh_priv_key', variable: 'SSH_PRIV_KEY')]) {
          sh("packer validate packer_gcp_nixos_${params.type}.json")
        }
      }
    }
    stage('Build image') {
      steps {
        withCredentials([string(credentialsId: 'project_id', variable: 'GCP_PROJECT_ID'), file(credentialsId: 'gcp_service_account_file', variable: 'GCP_SERVICE_ACCOUNT_FILE'), file(credentialsId: 'ssh_priv_key', variable: 'SSH_PRIV_KEY'), sshUserPrivateKey(credentialsId: 'github', keyFileVariable: 'SSH_KEY')]) {
          sh("packer build -force -color=false packer_gcp_nixos_${params.type}.json")
        }
      }
    }
    stage('Create and publish current image') {
      steps {
        withCredentials([string(credentialsId: 'project_id', variable: 'GCP_PROJECT_ID'), file(credentialsId: 'gcp_service_account_file', variable: 'GCP_SERVICE_ACCOUNT_FILE'), file(credentialsId: 'ssh_priv_key', variable: 'SSH_PRIV_KEY'), sshUserPrivateKey(credentialsId: 'github', keyFileVariable: 'SSH_KEY')]) {
          sh("git tag -a $BUILD_NUMBER -m 'Release $BUILD_NUMBER'")
          sh("git checkout master")
          sh("git pull origin master")
          sh("git merge origin/dev -m $BUILD_NUMBER")
          sh("git push origin master --tags")
          sh("git status")
        }
      }
    }
  }
}
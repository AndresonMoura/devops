def now
def tag

node {

  checkout scm
  now = new Date()
  tag = now.format("yyyyMMddHHmmss")

}

pipeline {

  agent any

  parameters {

    string(name: "ENVIRONMENT", defaultValue: params.ENVIRONMENT ?: "dev", description: "dev|test|homolog|production")
    string(name: "STACK_NAME", defaultValue: params.STACK_NAME ?: "", description: "stack name")
    string(name: "DOCKER_NETWORK_NAME", defaultValue: params.DOCKER_NETWORK_NAME ?: "", description: "Nome da rede no server remote")
    string(name: "SERVICE_NAME", defaultValue: params.SERVICE_NAME ?: "", description: "stack name")
    string(name: "IMAGE_REGISTRY", defaultValue: params.IMAGE_REGISTRY ?: "registry2.itbam.org.br:4000/devops/miscelaneos", description: "stack name")
    string(name: "DOCKERFILE_PATH", defaultValue: params.DOCKERFILE_PATH ?: "Jenkinsfile/Dockerfile", description: "Dockerfile path")
    string(name: "HOSTS_PATH", defaultValue: params.HOSTS_PATH ?: "deploy/hosts", description: "hosts")
    string(name: "DOCKER_COMPOSE_FILE", defaultValue: params.DOCKER_COMPOSE_FILE ?: "./docker-compose.yml", description: "Nome do arquivo do docker-compose")
    text(name: "DOCKER_COMPOSE_CONTENT", defaultValue: params.DOCKER_COMPOSE_CONTENT ?: "" , description: "Conteúdo do arquivo docker-compose. A tag ###IMAGE_REGISTRY### irá ser substituido pelo nome final da image gerada.")
	
    string(name: "REGISTRY_URL", defaultValue: params.REGISTRY_URL ?: 'http://registry2.itbam.org.br:4000', description: "Url do registry")
    string(name: "CREDENTIALS_ID", defaultValue: params.CREDENTIALS_ID ?: "jenkins-gitlab-local", description: "Jenkins credential to pull images from Git Registry")

    string(name: 'CONFIG_FILE_PATH', defaultValue: params.CONFIG_FILE_PATH ?: './.env', description: 'config file')
    text(name: 'CONTENT_CONFIG_FILE', defaultValue: params.CONTENT_CONFIG_FILE ?: '', description: 'contents of the environment variables in the file')


    // Ansible
    string(name: "ANSIBLE_DEPLOY_FILE", defaultValue: params.ANSIBLE_DEPLOY_FILE ?: "./playbook.yml", description: "Arquivo de deploy do ansible.")
    text(name: "ANSIBLE_HOSTS_CONTENT", defaultValue: params.ANSIBLE_HOSTS_CONTENT ?: "" , description: "Conetudo do arquivo hosts do ansible.")
    string(name: "ANSIBLE_HOSTS_FILE", defaultValue: params.ANSIBLE_HOSTS_FILE ?: "./hosts", description: "Arquivo hosts do ansible.")
    string(name: "ANSIBLE_DEPLOY_FILE_SRC", defaultValue: params.ANSIBLE_DEPLOY_FILE_SRC ?: "./docker-compose.yml", description: "Arquivo de deploy local.")
    string(name: "ANSIBLE_DEPLOY_FILE_DEST", defaultValue: params.ANSIBLE_DEPLOY_FILE_DEST ?: "/home/jenkins ...", description: "Arquivo de deploy no host remoto.")
    string(name: "ANSIBLE_CREDENCIAL_ID", defaultValue: params.ANSIBLE_CREDENCIAL_ID ?: "Credencial do host remoto no Jenkins", description: "Arquivo de deploy no host remoto.")

  }

  stages {

    stage("Prepare") {

      steps {

        sh script: """

cat > ${ANSIBLE_DEPLOY_FILE} <<EOF
- hosts: "SERVERS"
  any_errors_fatal: true
  tasks:
    - name: "Copy docker-compose to Swarm"
      copy:
        src: "${ANSIBLE_DEPLOY_FILE_SRC}"
        dest: "${ANSIBLE_DEPLOY_FILE_DEST}"
    - name: "Deploy stack"
      shell: docker stack deploy --compose-file ${ANSIBLE_DEPLOY_FILE_DEST} ${STACK_NAME} --with-registry-auth
EOF

cat > ${ANSIBLE_DEPLOY_FILE_SRC} <<EOF
${DOCKER_COMPOSE_CONTENT}
EOF

cat > ${ANSIBLE_HOSTS_FILE} <<EOF
${ANSIBLE_HOSTS_CONTENT}
EOF

sed -i "s|###IMAGE_REGISTRY###|${IMAGE_REGISTRY}:${tag}|g" "${ANSIBLE_DEPLOY_FILE_SRC}"

          """
      }

    }

    stage('Set environment variables') {
      steps {

        sh script: """

cat > ${CONFIG_FILE_PATH} <<EOF
${CONTENT_CONFIG_FILE}
EOF

        """

      }

    }


    stage("Build") {

      steps {
        sh "docker build -f ${DOCKERFILE_PATH} -t ${IMAGE_REGISTRY}:${tag} ."
      }

    }

    stage("Push") {

      steps {

            withDockerRegistry(credentialsId: "${CREDENTIALS_ID}", url: "${REGISTRY_URL}") {
                sh "docker push ${IMAGE_REGISTRY}:${tag}"
            }
        }

    }

    stage("Deploy") {

        steps {

            ansiblePlaybook colorized: true, credentialsId: "${ANSIBLE_CREDENCIAL_ID}", inventory: "${ANSIBLE_HOSTS_FILE}", playbook: "${ANSIBLE_DEPLOY_FILE}"
            
        }
    }

  }

}

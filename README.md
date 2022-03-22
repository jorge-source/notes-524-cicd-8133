# 524 - Integração e Entrega Continua com Git, Jenkins, Nexus e Sonar

Turma: 8133

Instrutor: Emerson Silva

Linkedln: https://www.linkedin.com/in/silvemerson/

Github: https://github.com/silvemerson

Medium: https://silvemerson.medium.com/


=================================================

Apostila do curso: https://moodle-prod.4linux.com.br/pluginfile.php/35274/mod_filewithwatermark/content/2/524-cicd_material_full_v14.pdf

Lab: https://github.com/4linux/524


Hardware Recommendations Jenkins
https://www.jenkins.io/doc/book/scaling/hardware-recommendations/


##### Vagrant Comandos

https://adamrushuk.github.io/cheatsheets/vagrant/


Reiniciar a maquina: vagrant reload NOME_MAQUINA


machines = {
  "cicd"       => {"memory"=>"1024", "cpus"=>"1", "ip" => "10" , "installDocker" => "yes"},
  "cicd-tools" => {"memory"=>"2048", "cpus"=>"1", "ip" => "20" , "installDocker" => "yes"},
  "homolog"    => {"memory"=>"512" , "cpus"=>"1", "ip" => "30" , "installDocker" => "yes"},
  "production" => {"memory"=>"512" , "cpus"=>"1", "ip" => "40" , "installDocker" => "yes"},
}



#####Roadmap DevOps

https://roadmap.sh/devops

##### Indicação de livro

https://www.amazon.com.br/Manual-DevOps-confiabilidade-organiza%C3%A7%C3%B5es-tecnol%C3%B3gicas/dp/8550802697

https://www.amazon.com.br/projeto-f%C3%AAnix-comemorativa-romance-neg%C3%B3cio/dp/8550814067/ref=sr_1_1?keywords=projeto+fenix&qid=1647388173&s=books&sprefix=projeto%2Cstripbooks%2C285&sr=1-1&ufe=app_do%3Aamzn1.fos.6d798eae-cadf-45de-946a-f477d47705b9


#####Gitea

vagrant up cicd-tools

vagrant ssh cicd-tools

sudo su

docker run -dti --name gitea --restart always -v gitea:/data -p 3000:3000 -p 2222:22 gitea/gitea:latest

docker ps

usuário: root
senha: 4linux
email: root@localhost.localdomain


===========================================================


Aula 02

Iniciando a máquina de CI/CD
 
vagrant up cicd

vagrant status

vagrant ssh cicd

sudo su


Primeira forma de Instalação via Docker-compose

mkdir jenkins

cd jenkins

nano docker-compose.yml

docker-compose.yml

###########
version: "3"

networks:
   jenkins:
     external: false
 
services:
  jenkins-ci-server:
    user: root
    container_name: jenkins-ci-server
    image: jenkins/jenkins:alpine

    environment:
      - USER_UID=1000
      - USER_GID=1000
    restart: always
    networks:
      - jenkins
    volumes:
      - /var/jenkins_home
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:rw
      - /usr/bin/docker:/usr/bin/docker
    ports:
      - "8080:8080"
##########

docker-compose config


docker-compose up -d

docker-compose ps

http://192.168.56.10:8080/
 
docker exec -it jenkins-ci-server cat /var/jenkins_home/secrets/initialAdminPassword



Configuração usuário Jenkins:

user: root
senha: 4linux
Full name: root
email: root@localhost

Jenkins With Microsoft Teams
https://dzone.com/articles/configure-jenkins-notifications-with-microsoft-tea


Plugins a serem instalados:

• SonarQube Scanner
• Nexus Platform
• Pipeline Utility Steps
• Gitea
• Gogs
• Locale

https://miro.medium.com/max/1400/1*g1J31rHks_8L-35j1rsD3A.png

https://github.com/alexguzun/jenkins-pipeline-gitflow-maven/blob/master/Jenkinsfile


===== Sonarqube

docker run -dti --name sonarqube \
 -v sonarqube_conf:/opt/sonarqube/conf \
 -v sonarqube_extensions:/opt/sonarqube/extensions \
 -v sonarqube_logs:/opt/sonarqube/logs \
 -v sonarqube_data:/opt/sonarqube/data \
 --restart always -p 9000:9000 sonarqube:latest  


http://192.168.56.20:9000/

user: admin
senha: admin

https://docs.sonarqube.org/latest/analysis/coverage/

=================================================================================

## AULA 3



Instalando jenkins de maneira manual no Debian 10


### Primeiro vamos parar container do jenkins

```shell
# docker container stop idconatiner-jenkins
```

### Adicionando chave do jekins ao apt 

```shell
# wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
```

### Repo jenkins

```shell
echo deb https://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list
```


### Atualizar lista de REPOS

```shell
# apt update
```

### Instalação do JDK 11

```shell
# apt install openjdk-11-jdk
```

### Primeiro pipeline-declarativo

```shell
pipeline {
    
    agent any
    
    stages {
        
        stage('OS Release'){
            steps {
                sh 'cat /etc/*release'
                echo "$WORKSPACE"
                echo "$BUILD_ID"
            }
        }
    }
        
}

```
### Gitea Deploy

```shell
docker run -dti --name gitea --restart always -v gitea:/data -p 3000:3000 -p 2222:22 gitea/gitea:latest
```
### Options pipeline declarativo

```
pipeline {
    
    agent any
    
    options{
        timeout(time: 5, unit: 'SECONDS')
        timestamps()
    }
    
    stages {
        
        stage('OS Release'){
            steps {
                sh 'cat /etc/*release'
                echo "$WORKSPACE"
                echo "$BUILD_ID"
            }
        }
        
        stage('Git Checkout do aula_git'){
            steps{
                git 'http://192.168.56.10:3000/root/aula_git.git'
            }
            
        }
    }
        
}
```



### Trabalhando com Jenkinsfile com docker

```groovy
pipeline{
    agent any

    environment {
            IMAGE_NAME="simple-python-flask"
        }


    stages{
        
        stage('Image Build'){
            steps{
                script{
                    image = docker.build("$IMAGE_NAME")
                }
            }
        }

    }


}

```

### Jenkinsfile stage de testes unitários 

```groovy

pipeline{
    agent any

    environment {
            IMAGE_NAME="simple-python-flask"
        }


    stages{
        
        stage('Image Build'){
            steps{
                script{
                    image = docker.build("$IMAGE_NAME")
                }
            }
        }

        stage('Running Unit Test'){
            steps{
                script{
                    image.inside("-v ${WORKSPACE}:/simplePythonApplication"){
                        sh "nosetests --with-xunit --with-coverage --cover-package=project test_users.py"

                    }
                }
            }

        }


    }


}

```


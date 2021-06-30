### install docker

1. sudo apt-get remove docker docker-engine docker.io containerd runc
383. sudo dpks --configure a
384. sudo dpkg --configure -a
385. sudo apt update && sudo apt upgrade
386. sudo apt-get remove docker docker-engine docker.io containerd runc
387. sudo apt-get remove docker docker.io containerd runc
388. sudo add-apt-repository \\n\n     "deb [arch=amd64] https://download.docker.com/linux/ubuntu \\n\n     bionic \\n\n     test"
389. sudo add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu  bionic  test"
390. sudo apt-get install -y docker-ce
391. clear
392. sudo apt install apt-transport-https ca-certificates curl software-properties-common
395. sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
396. sudo apt update
397. sudo apt install docker-ce
398. sudo systemctl stop docker.service
399. sudo systemctl start docker.service
400. sudo systemctl status docker.service
401. clear
402. sudo docker ps
403. sudo docker run -p 8081:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker jenkins/jenkins:lts
406. sudo docker volume ls
412. sudo docker ps
413. sudo docker exec -u 0 -it {id}
414. sudo docker exec -u 0 -it {id} bash

-----------------------------------------------------------------------------------------
### // docker basic run command
docker run -p 8081:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts

### // beast sys error so -p 8081:8080

docker ps <br>
docker exec -u 0 -it {id} bash

### // jenkins intial admin password
cat /var/jenkins_home/secrets/initialAdminPassword <br>
docker volume inspect jenkins_home

### // to find ubuntu distribution : 
cat /etc/issue

### // giving permission to docker socket <br>
sudo chmod 666 /var/run/docket.sock <br>
sudo chmod 777 /var/run/docker.sock {worked for me}

### // run docker with mount options <br>
sudo docker run -p 8081:8080 -p 50000:50000 -d -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker jenkins/jenkins:lts

--------------------------------------------------------------------------------
### Pushing Image to docker container

#### {To be done in jenkins  -> configure}

docker build -t happyfacep/my-repo:jma-1.0 .

docker login -u $USERNAME -p $PASSWORD 

(or)

echo $PASSWORD | docker login -u $USERNAME --password-stdin


docker push happyfacep/my-repo:jma-1.0

--------------------------------------------------------------------------------
### Freestyle to Pipeline Job

#### Groovy SandBox:
    * Means can only execute /use only limited number of groovy functions such that you don't need approval from jenkins admin

### Required fields 
    * pipeline
    * agent
    * stages
        * stage types "iterator"
            * steps 

_______________________________________________________________________________
### Jenkinsfile Syntax

* `post` attribute , executed after all stages are executed, similar to `finally` in java code.
```
post {
    always{

    }
    failure{

    }
}
```
________________________________________________________________________________
### ENVIRONMENT VARIABLES
* where can we find the environmental variables
* sample url : http://localhost:8081/env-vars.html/
* how to create custom environmental variables 
    * insert environment {} after agent any 

### HOW TO USE ENVIRONMENTAL VARIABLES

```

pipeline {
	agent any 
	environment {
		NEW_VERSION = '1.3.0'
	}
	stages {
		stage("build") {
			// when {
			// 	expression {
			// 		BRANCH_NAME = 'dev' && CODE_CHANGES == true
			// 	}
			// }
			steps {
				echo 'builidng!'
				echo "building version ${NEW_VERSION}"
			}
		}
		stage("test") {
			// when {
			// 	expression {
			// 		BRANCH_NAME = 'dev' || BRANCH_NAME = 'master'
			// 	}
			// }
			steps {
				
			}
		}
		stage("build") {
			steps {
				
			}
		}
	}
}

```
_______________________________________________________________
### HOW TO USE CONDITION
```
pipeline {
	agent any 
	stages {
		stage("build") {
			when {
				expression {
					BRANCH_NAME = 'dev' && CODE_CHANGES == true
				}
			}
			steps {
				echo 'builidng!'
				echo "building version ${NEW_VERSION}"
			}
		}
    }
}

```
_____________________________________________________________________________
### getting password using jenkins global configuration
```
pipeline {
	agent any 
	environment {
		NEW_VERSION = '1.3.0'
		SERVER_CREDENTIALS = credentials('server-credentials') // credentials plugin
	}
	stages {
		stage("build") {
			steps {
				echo 'builidng!'
				echo "building version ${NEW_VERSION}"
			}
		}
		stage("test") {
			steps {
				
			}
		}
		stage("deploy") {
			steps {
				echo "credentials is ${SERVER_CREDENTIALS}"
				sh "${SERVER_CREDENTIALS}"

				// to use directly in stage
				// with credentials wrapper
				withCredentials([
					usernamePassword(credentials:'server-credentials',usernameVariable: USER,passwordVariable:PWD)
				]) {
					sh "somescript ${USER} ${PWD}"
				}
			}
		}
	}
}

```
____________________________________________
### Access Build Tools for your projects
```
pipeline {
	agent any 
	tools {
		maven 'Maven'

		// tool name taken from global tool configuration
		// gradle
		// jdk
	}
	stages {
		stage("build") {
			steps {
				echo "building"
                //sh "maven commands"
			}
		}
		stage("test") {
			steps {
				echo "testing"
			}
		}
		stage("deploy") {
			steps {
				echo "deployment"
			}
		}
	}
}

```
____________________________________________
### Parameters in jenkins 
```
pipeline {
	agent any 
	parameters {
		string(name: 'VERSION'),defaultValue:'',description:'version to be deployed on prod')
		choice(name: 'VERSION'),choices:['1.1.0','1.2.0'],description:'')
		booleanParam(name:'executeTests',defaultValue:true,description:'')
	}
}

```

### Working Example
```
pipeline {
	agent any 
	parameters {
		choice(name: 'VERSION', choices: ['1.1.0', '1.2.0', '1.3.0'], description: '')
        booleanParam(name: 'executeTests', defaultValue: true, description: '')
	}
	stages {
		stage("build") {
			steps {
				echo "building"
			}
		}
		stage("test") {
			when {
				expression {
					params.executeTests == true
				}
			}
			steps {
				echo "testing"
			}
		}
		stage("deploy") {
			steps {
				echo "deployment"
				echo "deploy version ${params.VERSION}"
			}
		}
	}
}

```
_____________________________________________________________
### Usage of External Script like groovy , shell script

* here let us see an basic example of how to use groovy script 
with jenkins file 
### Jenkinsfile 
```
def gv

pipeline {
	agent any 
	parameters {
		choice(name: 'VERSION', choices: ['1.1.0', '1.2.0', '1.3.0'], description: '')
        booleanParam(name: 'executeTests', defaultValue: true, description: '')
	}
	stages {
		stage("init") {
			steps {
				script {
					gv = load "script.groovy"
				}
			}
		}

		stage("build") {
			steps {
				script {
					gv.buildApp()
				}
			}
		}
		stage("test") {
			when {
				expression {
					params.executeTests == true
				}
			}
			steps {
				script {
					gv.testApp()
				}
			}
		}
		stage("deploy") {
			steps {
				script {
					gv.deployApp()
				}
			}
		}
	}
}

```

### script.groovy
```
def buildApp() {
    echo "building!"
}

def testApp() {
    echo "testing"
}

def deployApp(){
    echo "deployment"
	echo "deploy version ${params.VERSION}"
}

return this
```

______________________________________________________________________________
### Input Parameter in jenkins file 
* single or multiple inputs are available 

### Jenkinsfile
* using jenkins parameters 
```
def gv

pipeline {
	agent any 
	parameters {
		choice(name: 'VERSION', choices: ['1.1.0', '1.2.0', '1.3.0'], description: '')
        booleanParam(name: 'executeTests', defaultValue: true, description: '')
	}
	stages {
		stage("init") {
			steps {
				script {
					gv = load "script.groovy"
				}
			}
		}

		stage("build") {
			steps {
				script {
					gv.buildApp()
				}
			}
		}
		stage("test") {
			when {
				expression {
					params.executeTests == true
				}
			}
			steps {
				script {
					gv.testApp()
				}
			}
		}
		stage("deploy") {
			input {
				message "select the environment to deploy to "
				ok "Done"
				parameters {
					choice(name: 'ENV1', choices: ['dev','staging','prod'], description: '')
                    choice(name: 'ENV2', choices: ['dev','staging','prod'], description: '')
				}
			}
			steps {
				script {
					gv.deployApp()
					echo "Deploying to ${ENV1}"
                    echo "Deploying to ${ENV2}"
				}
			}
		}
	}
}


```

* using groovy script parameters type

```

def gv

pipeline {
	agent any {
        stages {
            stage("deploy") {
                steps {
                    script {
                        env.ENV = input message: "Select the environment to deploy to",ok:"Done", parameters: [choice(name: 'ONE', choices: ['dev','staging','prod'], description: '')]
                        gv.deployApp()
                        echo "Deploying to ${ENV}"
                    }
                }
            }
        }
    }
}

```

### script.groovy 

```
def buildApp() {
    echo "building!"
}

def testApp() {
    echo "testing"
}

def deployApp(){
    echo "deployment"
	echo "deploy version ${params.VERSION}"
}

return this
```
______________________________________________________________________________
### Jenkins Complete Pipeline

>Example: Build Java App -> Build Image -> Push to private repo

### Simple pipeline without groovy script

### Jenkinsfile
```
pipeline {
	agent any 
	tools {
		maven 'maven-3.8'
	}
	stages {
		stage("build jar") {
			steps {
				script {
					echo "building maven app!!!"
					sh "mvn package"
				}
			}
		}

		stage("build image") {
			steps {
				script {
					echo "building docker image!!!"
					withCredentials([usernamePassword(credentialsId: '4ee958bd-1d7b-4e5b-abcc-4663312dacc6',passwordVariable:'PASS',usernameVariable:'USER')]) {
						sh 'docker build -t happyfacep/my-repo:jma-2.0 .'
						sh "echo $PASS | docker login -u $USER --password-stdin"
						sh "docker push happyfacep/my-repo:jma-2.0"
					}
				}
			}
		}
		
		stage("deploy") {
			steps {
				script {
					echo "deploying the app!!!"
				}
			}
		}
	}
}
```

### Simple pipeline with groovy script 

### Jenkinsfile
```
def gv

pipeline {
	agent any 
	tools {
		maven 'maven-3.8'
	}
	stages {
		stage("init") {
			steps {
				script {
					gv = load "script.groovy"
				}
			}
		}
		stage("build jar") {
			steps {
				script {
					gv.buildJar()
				}
			}
		}

		stage("build image") {
			steps {
				script {
					gv.buildDocker()
				}
			}
		}
		
		stage("deploy") {
			steps {
				script {
					gv.deployApp()
				}
			}
		}
	}
}
```

### script.groovy
```
def buildJar() {
   echo "building maven app!!!"
   sh "mvn package"
}

def buildDocker() {
    echo "building docker image!!!"
    withCredentials([usernamePassword(credentialsId: '4ee958bd-1d7b-4e5b-abcc-4663312dacc6',passwordVariable:'PASS',usernameVariable:'USER')]) {
        sh 'docker build -t happyfacep/my-repo:jma-2.0 .'
        sh "echo $PASS | docker login -u $USER --password-stdin"
        sh "docker push happyfacep/my-repo:jma-2.0"
    }
}

def deployApp(){
    echo "deploy app!!!"
}

return this
```

______________________________________________________________________________________
### Multibranch pipeline

> Multibranch Pipeline -> dynamically create pipelines for branches


### Branch based logic for muli-branch in Jenkinsfile

> Pipeline example to execute only on master branch in multi-branch pipeline

```
pipeline {
	agent any 
	stages {
		stage('test') {
			steps {
				script {
					echo "Testing !!!"
					echo "Executing pipeline for branch $BRANCH_NAME"
				}
			}
		}
		stage('build') {
			when {
				expression {
					BRANCH_NAME == 'master'
				}
			}
			steps {
				script {
					echo "Building!!!"
				}
			}
		}		
		stage("deploy") {
			steps {
				script {
					echo "Deploy!!!"
				}
			}
		}
	}
}
```
______________________________________________________________________________________
### Credentials in depth

> Credentials Plugin to store and manage centrally.

> System credentials available only on server

> Global credentials available on both locally and server

> can also be able to create credentials for seperate project / pipeline

_______________________________________________________________
### Jenkins shared library

> example scenario: ten microservices concept

### Shared Library
* extension to the pipeline
* has own repository
* written in groovy
* referenced shared logic in Jenkinsfile


### Steps
* repository
* write groovy script
* make shared library available globally
* use this library in the Jenkinsfile to extend the pipeline


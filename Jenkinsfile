pipeline {
    agent none
    environment {
        IMAGE_NAME = 'xeliol/django_demo'
        HUB_CRED_ID = 'Docker_Marinin'
        PROJECT_DIR = 'common_marinin_django'
    }
    stages {
        stage("deps") {
            agent {
                docker {
                    image 'python:latest'
                    args '-u root -v ${WORKSPACE}/pipenv:/root/.local'
                }
            }
            steps {
                sh 'pip install --user -r requirements.txt'
            }
        }
        stage("test") {
            agent {
                docker {
                    image 'python:latest'
                    args '-u root -v ${WORKSPACE}/pipenv:/root/.local'
                }
            }
            steps {
                sh 'python -m coverage run manage.py test'
            }
        }
        stage("report") {
            agent {
                docker {
                    image 'python:latest'
                    args '-u root -v ${WORKSPACE}/pipenv:/root/.local'
                }
            }
            steps {
                sh 'python -m coverage report'
            }
        }
        stage("build") {
            agent any
            steps {
                sh 'docker build . -t ${IMAGE_NAME}:${GIT_COMMIT}'
            }
        }
        stage("push") {
            agent any
            steps {
                withCredentials([usernamePassword(credentialsId: "${HUB_CRED_ID}",
                usernameVariable: 'HUB_USERNAME', passwordVariable: 'HUB_PASSWORD')]) {
                    sh 'docker login -u ${HUB_USERNAME} -p ${HUB_PASSWORD}'
                    sh 'docker push ${IMAGE_NAME}:${GIT_COMMIT}'
                    sh 'docker push ${IMAGE_NAME}:latest'
                }  
            }
        }
        stage("deploy") {
        	agent any
        	steps {
        		withCredentials(
        			[
        				string(credentialsId: "production_ip", variable: "SERVER_IP"),
        				sshUserPrivateKey(credentialsId: "production_key", keyFileVariable: "SERVER_KEY", usernameVariable: "SERVER_USERNAME")
        			]
        		) 
        		{
        			sh 'ssh -i ${SERVER_KEY} ${SERVER_USERNAME}@${SERVER_IP} mkdir -p ${PROJECT_DIR}'
                    sh 'scp -i ${SERVER_KEY} docker-compose.yaml ${SERVER_USERNAME}@${SERVER_IP}:${PROJECT_DIR}'
                    sh 'ssh -i ${SERVER_KEY} ${SERVER_USERNAME}@${SERVER_IP} docker compose -f ${PROJECT_DIR}/docker-compose.yaml pull'
                    sh 'ssh -i ${SERVER_KEY} ${SERVER_USERNAME}@${SERVER_IP} docker compose -f ${PROJECT_DIR}/docker-compose.yaml up -d'
        			
        		}
        	}
        }
        stage("config proxy") {
        	agent any
        	steps {
        		withCredentials(
        			[
        				string(credentialsId: "production_ip", variable: "SERVER_IP"),
        				sshUserPrivateKey(credentialsId: "production_key", keyFileVariable: "SERVER_KEY", usernameVariable: "SERVER_USERNAME")
        			]
        		) 
        		{
		    		sh 'scp -i ${KEY_FILE} paramonov.prod.mshp-devops.conf ${USERNAME}@${PROD_IP}:nginx'
		    		sh 'ssh -i ${KEY_FILE} ${USERNAME}@${PROD_IP} sudo systemctl reload nginx'
		    	}
        	}
        }
    }
}

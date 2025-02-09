pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker tag:')
    }
    
    tools {
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/leroyjenkin/DevOps-MultiTierBankAppCI-pipeline.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o fs.html"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=bankapp -Dsonar.projectName=bankapp -Dsonar.java.binaries=target'''
                }
            }
        }
            
        stage('Building and Publish Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'maven-settings-devopsshack', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }

        stage('Docker Build & Tag') {
                steps {
                        script {
                                withDockerRegistry(credentialsId: 'docker-cred') {
                                        sh "docker build -t adijaiswal/bankapp:${params.DOCKER_TAG} ."
                                }
                        }
                }
        }

        stage('Docker Image Scan') {
                steps {
                        sh "trivy image --format table -o dimage.html adijaiswal/bankapp:${params.DOCKER_TAG}"
                }
        }

        stage('Docker Push') {
                steps {
                        script {
                                withDockerRegistry(credentialsId: 'docker-cred') {
                                        sh "docker push adijaiswal/bankapp:${params.DOCKER_TAG}"
                                }
                        }
                }
        }
        stage('Update YAML Manifest in Other Repo') {
                steps {
                        script {
                                withCredentials([gitUsernamePassword(credentialsId: 'git-cred', gitToolName: 'Default')]) {
                                        sh '''
                     			      	# Clone the repo
                     	                        git clone https://www.github.com/jaiswaladi246/Multi-Tier-BankApp-CD.git
                                   	        cd Multi-Tier-BankApp-CD

                       	                        # List files to confirm the presence of bankapp-ds.yml
                                                ls -l bankapp

                                                # Get the absolute path for the current directory
                                                repo_dir=$(pwd)

                                                # Use the absolute path for sed
                                                sed -i 's|image: adijaiswal/bankapp:.*|image: adijaiswal/bankapp:${DOCKER_TAG}|' ${repo_dir}/bankapp/bankapp-ds.yml
                                	'''


                        	        // Confirm the change
                          	        sh '''
                         	   	     echo "Updated YAML file contents:"
				   	     cat Multi-Tier-BankApp-CD/bankapp/bankapp-ds.yml
                          	        '''

				        // Configure Git for committing changes and pushing
        				sh '''
        			             cd Multi-Tier-BankApp-CD   # Ensure you are inside the cloned repo
        			             git config user.email "office@devopsshack.com"
       			                     git config user.name "DevOps Shack"
       				        '''

        				// Commit and push the updated YAML file back to the other repository
        				sh '''
        			            cd Multi-Tier-BankApp-CD
        			            ls
        			            git add bankapp/bankapp-ds.yml
        			            git commit -m "Update image tag to ${DOCKER_TAG}"
        			            git push origin main
        				'''
				}
                        }
                }
}



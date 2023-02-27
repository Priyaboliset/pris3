pipeline {
    agent {
        label 'component_library'
      }
    
        stage('provision s3 module') {
            steps {
			    script{
					withCredentials([usernamePassword(credentialsId: '7cebc952-30c8-40ee-a965-2fef84e81168', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {    
							sh '''
							git config --global credential.helper "!f() { echo username=$GIT_USERNAME; echo password=$GIT_PASSWORD; }; f"
				 			git clone https://innersource.accenture.com/scm/piab/acc-aws-s3-module.git
							cd acc-aws-s3-module
							terraform init
							terraform plan -out=tfplan.binary
							terraform show -json tfplan.binary > plan.json
							'''
						}
                    stash includes: 'plan.json', name: 'plan_json'
                } 
            }
	    }
        stage('infracost') {
            agent {
                docker {
                    image 'infracost/infracost:ci-0.9'
                    args "--user=root --entrypoint=''"
                }
            }

            environment {
                INFRACOST_API_KEY = credentials('jenkins-infracost-api-key')
            }

            steps {
                unstash 'plan_json'
                sh 'infracost breakdown --path plan.json --format json --out-file infracost.json'

            }
        }
	}

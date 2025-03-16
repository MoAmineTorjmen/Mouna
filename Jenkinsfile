pipeline {
  tools {
    git "git"
    gradle "gradle"
    nodejs "node"
  }
  agent {
    label 'master'
  }
  stages {
	stage('Clean Workspace') {
		agent {
			node {
			  label 'agent1'
			}
		}
		steps {
			// Clean before build
			echo "Begin cleanning workspace before build"
			cleanWs()			
		}
	}
	
	stage('SonarQube analysis') {
		parallel {
			stage('Backend Analysis') {
				agent {
					node {
						label 'agent1'
					}
				}
				steps {
					withSonarQubeEnv(installationName: 'sonarqube', credentialsId: 'sonarqube') {
						dir("accelengine/backend") {
							sh 'gradle sonarqube -Dsonar.projectName="GA backend" -Dsonar.projectKey="ga-backend"'
						}
					}
				}
			}

			stage('Frontend Analysis') {
				agent {
					node {
						label 'agent1'
					}
				}
				steps {
					dir("accelengine") {
						sh '''
						npm install sonar-scanner --save-dev --force;
						npm run sonar
						'''
					}
				}
			}
		}
	}

   	stage("Quality gate") {
		steps {
			dir("accelengine/backend") {
				catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
					waitForQualityGate abortPipeline: true
				}
			}
		}
    }
	
    stage('Start container') {
      agent {
        node {
          label 'agent1'
        }
      }
      tools {
        git "git"
      }
      steps {
       git branch: 'develop',
          credentialsId: '884a193f-afb5-4641-a497-452dd0507415',
          url: 'http://ftpdisco:3001/GA/GA.git'
      }
    }
  }
}

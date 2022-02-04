/pipeline {
	agent none
	 stages{
		 stage("worker-build"){
		 	when{
			 changeset "**/worker/**"
			 }
			 agent{
				 docker{
				 image 'maven:3.6.1-jdk-8-slim'
				 args '-v $HOME/.m2:/root/.m2'
				 }
			 }
			 steps{
				 echo 'Compiling worker app..'
				 dir('worker'){
				 sh 'mvn compile'
				 }
			 }
			 }
		 stage("worker-test"){
		 	when{
			changeset "**/worker/**"
	 		}
			agent{
 				docker{
 				image 'maven:3.6.1-jdk-8-slim'
 				args '-v $HOME/.m2:/root/.m2'
 				}
 			}
 			steps{
				echo 'Running Unit Tets on worker app..'
 				dir('worker'){
 				sh 'mvn clean test'
 				}
 			}
 			}		
 		stage("worker-package"){
 			when{
 			branch 'master'
 			changeset "**/worker/**"
 			}
 			agent{
 				docker{
 				image 'maven:3.6.1-jdk-8-slim'
 				args '-v $HOME/.m2:/root/.m2'
 				}
 			}
 			steps{
 				echo 'Packaging worker app'
 				dir('worker'){
 				sh 'mvn package -DskipTests'
 				archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
				}
			}	
			}
 		stage('worker-docker-package'){
 			agent any
 			when{
 			changeset "**/worker/**"
 			branch 'master'
 			}
 		steps{
 			echo 'Packaging worker app with docker'
 			script{
 				docker.withRegistry('https://index.docker.io/v1/', 'nmoiseykin-docker'){
 				workerImage = docker.build("nmoiseykin/worker:v${env.BUILD_ID}", "./worker")
 				workerImage.push()
 				workerImage.push("${env.BRANCH_NAME}")
 				workerImage.push("latest")
 				}		
 			}
 			}
 			}       
		stage('result-build'){
            agent{
              docker{
                image 'node:8.16.0-alpine'
                }
            }

            when{
              changeset "**/result/**"
            }
            steps{
                echo 'Compiling result app'
                dir('result'){
                  sh 'npm install'
                }
            }
        }
		stage('result-test'){
           agent{
              docker{
                image 'node:8.16.0-alpine'
                }
            }
        when{
              changeset "**/result/**"
            }
            steps{
                echo 'Running Unit Tests on result app'
                dir('result'){
                  sh 'npm install'
                  sh 'npm test'
                }
            }
        }
      stage('result-docker-package'){
                        agent any
                        when{
                        changeset "**/result/**"
                        branch 'master'
                        }
                steps{
                        echo 'Packaging result app with docker'
                        script{
                                docker.withRegistry('https://index.docker.io/v1/', 'nmoiseykin-docker'){
                                workerImage = docker.build("nmoiseykin/result:v${env.BUILD_ID}", "./result")
                                workerImage.push()
                                workerImage.push("${env.BRANCH_NAME}")
                                workerImage.push("latest")
                                }
                        }
                        }
                        }

		stage('vote-build'){
            agent{
              docker{
                image 'python:2.7.16-slim'
                args '--user root'
                }
        }
            when{
              changeset "**/vote/**"
            }
            steps{
                echo 'Compiling vote app'
                dir('vote'){
                  sh 'pip install -r requirements.txt'
                }
            }
        }



	stage('vote-test'){
            agent{
              docker{
                image 'python:2.7.16-slim'
                args '--user root'
                }
            }
              when{
              changeset "**/vote/**"
            }
            steps{
                echo 'Running Unit Tests on vote app'
                dir('vote'){
                  sh 'pip install -r requirements.txt'
                  sh 'nosetests -v'
                }
            }
        }

	stage('vote-intergration'){
            agent any
              when{
              changeset "**/vote/**"
	      branch 'master'
            }
            steps{
                echo 'Running Integration Tests on vote app'
                dir('vote'){
                  sh 'integrtion_test.sh'
                }
            }
        }

	stage('vote-docker-package'){
                        agent any
                        when{
                        changeset "**/vote/**"
                        branch 'master'
                        }
                steps{
                        echo 'Packaging vote app with docker'
                        script{
                                docker.withRegistry('https://index.docker.io/v1/', 'nmoiseykin-docker'){
                                workerImage = docker.build("nmoiseykin/vote:v${env.BUILD_ID}", "./vote")
                                workerImage.push()
                                workerImage.push("${env.BRANCH_NAME}")
                                workerImage.push("latest")
                                }
                        }
                        }
                        }



    stage('Sonarqube') {
      agent any
 /*     when{
        branch 'master'
      }
*/
      tools {
        jdk "JDK11" // the name you have given the JDK installation in Global Tool Configuration
      }

      environment{
        sonarpath = tool 'SonarScanner'
      }

      steps {
            echo 'Running Sonarqube Analysis..'
            withSonarQubeEnv('sonar-example-vote') {
              sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
            }
      }
    }


    stage("Quality Gate") {
        steps {
            timeout(time: 1, unit: 'HOURS') {
                // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                // true = set pipeline to UNSTABLE, false = don't
                waitForQualityGate abortPipeline: true
            }
        }
    }


 	}
	 post{
		 always{
		 echo 'Building multibranch pipeline for worker is completed..'
 		}
 	}
}
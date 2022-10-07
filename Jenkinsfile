pipeline {
	    agent any
	

	    // Declare Environment Variables
	    environment {
	        	//Orchestrator Services
				      UIPATH_ORCH_URL = "https://cloud.uipath.com/"
	            UIPATH_ORCH_LOGICAL_NAME = "SocMediaBot"
	            UIPATH_ORCH_TENANT_NAME = "Default"
	            UIPATH_ORCH_FOLDER_NAME = "SocialMediaBots" // replace this variable with your UiPath folder name
			        EXAPP_APPID = "aab0a281-2be3-45ac-af92-2858941e858a"
			        EXAPP_SCOPES = "OR.Assets OR.BackgroundTasks OR.Execution OR.Folders OR.Jobs OR.Machines.Read OR.Monitoring OR.Robots.Read OR.Settings.Read OR.TestSetExecutions OR.TestSets OR.TestSetSchedules OR.Users.Read"
			        EXAPP_IDENTITYURL = "https://cloud.uipath.com/aitraining"
	    }
	

	    stages {
	        stage('Preparing'){
	            steps {
	                echo "Jenkins Home ${env.JENKINS_HOME}"
	                echo "Jenkins URL ${env.JENKINS_URL}"
	                echo "Jenkins JOB Number ${env.BUILD_NUMBER}"
	                echo "Jenkins JOB Name ${env.JOB_NAME}"
	                echo "GitHub BranchName ${env.BRANCH_NAME}"
	                checkout scm
		    	}
	        }	
	    
			// UiPath Run Tests
	        stage('Unit Tests') {
	            steps {
	            echo 'Testing the workflow...'
				UiPathTest (
					testTarget: [$class: 'TestProjectEntry', testProjectPath: "${WORKSPACE}", environments: ""],
					orchestratorAddress: "${UIPATH_ORCH_URL}",
					orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
					folderName: "${UIPATH_ORCH_FOLDER_NAME}",
					timeout: 10000,
					traceLevel: 'None',
					testResultsOutputPath: '',
          credentials: ExternalApp(accountForApp: "${UIPATH_ORCH_LOGICAL_NAME}", applicationId: "${EXAPP_APPID}", applicationScope: "${EXAPP_SCOPES}", identityUrl: "${EXAPP_IDENTITYURL}"),
					//credentials: ExternalApp(accountForApp: "${UIPATH_ORCH_LOGICAL_NAME}", applicationId: "${EXAPP_APPID}", applicationSecret: 'ExApp_aitraining_Mai_TestSuiteTraining', applicationScope: "${EXAPP_SCOPES}", identityUrl: "${EXAPP_IDENTITYURL}"),
					parametersFilePath: ''
				)
	            }
			}    
		    
			// UiPath Pack
	        stage('Build Process') {
				when {
					expression {
						currentBuild.result == null || currentBuild.result == 'SUCCESS' 
					}
				}				
				steps {
					echo "Building package with ${WORKSPACE}"
					UiPathPack (
						outputPath: "Output\\${env.BUILD_NUMBER}",
						projectJsonPath: "project.json",
						outputType: "Process",
						// runWorkflowAnalysis: true,
						version: AutoVersion(),
						useOrchestrator: false,
						traceLevel: 'Information'
					)
				}
	        }
			
	        // UiPath Deploy
	        stage('Deploy Process') {
				when {
					expression {
						currentBuild.result == null || currentBuild.result == 'SUCCESS' 
						}
				}
				steps {
	                echo 'Deploying process to orchestrator...'
	                UiPathDeploy (
	                		packagePath: "Output\\${env.BUILD_NUMBER}",
	                		orchestratorAddress: "${UIPATH_ORCH_URL}",
	                		orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
	               			folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                      environments: "env",
                      createProcess: true,
                      credentials: credentials: ExternalApp(accountForApp: "${UIPATH_ORCH_LOGICAL_NAME}", applicationId: "${EXAPP_APPID}", applicationSecret: 'ExApp_aitraining_Mai_TestSuiteTraining', applicationScope: "${EXAPP_SCOPES}", identityUrl: "${EXAPP_IDENTITYURL}"),
          //					credentials: ExternalApp(accountForApp: "${UIPATH_ORCH_LOGICAL_NAME}", applicationId: "${EXAPP_APPID}", applicationSecret: 'ExApp_aitraining_Mai_TestSuiteTraining', applicationScope: "${EXAPP_SCOPES}", identityUrl: "${EXAPP_IDENTITYURL}"),
                      traceLevel: 'Information',
					          	entryPointPaths: 'Main.xaml'
					)
				}
			}
	    }
	

	    // Options
	    options {
	        // Timeout for pipeline
	        timeout(time:80, unit:'MINUTES')
	        skipDefaultCheckout()
	    }


	    // post actions
	    post {
	        success {
	            echo 'Deployment has been completed!'
	        }
	        failure {
	          echo "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.JOB_DISPLAY_URL})"
	        }
			always {
				// clean up workspace
				deleteDir()
	        }
	    }
}

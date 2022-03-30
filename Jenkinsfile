pipeline {
agent {
   label 'jenkinsNode'
}

environment {
    config = readJSON file: 'config.json'
	
	GitSource = "${config.git_repo.URL}"
	GitCred = "${config.git_repo.Cred}"
	Branch = "${config.git_repo.branch}"
	
	projectName = "${config.projectDetails.Name}"
	projectLanguage = "${config.projectDetails.language}"
	projectVersion = "${config.projectDetails.version}"
	reportsPath = "${config.projectDetails.reportsPath}"
	
	languageVersion = "${config.build.version}"
	buildTool = "${config.build.buildTool}"
	
	//For Ant Build
	antBuildFile = "${config.build.antBuildFile}"
	antCompileTarget = "${config.build.antCompileTarget}"
	antUnitTestTarget = "${config.build.antUnitTestTarget}"
	
	//dotNet build
	dotNetSolutionPath = "${config.build.dotNetSolutionPath}"
	dotNetAppType = "${config.build.dotNetAppType}"
	
	//cmake for C CPP projects
	cmakeInstallation = "${config.build.cmakeInstallation}"
	cmakeBuildDir = "${config.build.cmakeBuildDir}"
	cmakeBuildType = "${config.build.cmakeBuildType}"
	cmakeGenerator = "${config.build.cmakeGenerator}"
	
	jiraSite = "${config.jira.site}"
	jiraProjectKey = "${config.jira.projectKey}"
	jiraIssueType = "${config.jira.issueType}"
	jiraAutoRaiseIssue = "${config.jira.autoRaiseIssue}"
	jiraAutoResolveIssue = "${config.jira.autoResolveIssue}"
	jiraAutoUnlinkIssue = "${config.jira.autoUnlinkIssue}"
	
	skipUnitTestCases = "${config.unitTestCases.skipUnitTestCases}"
	skipPublishingChecks = "${config.unitTestCases.skipPublishingChecks}"
	jUnitTestResultsPath = "${config.unitTestCases.jUnitTestResultsPath}"
	pythonUnitTestFileLocation = "${config.unitTestCases.pythonUnitTestFileLocation}"
	dotNetUnitTestSrcPath = "${config.unitTestCases.dotNetUnitTestSrcPath}"
	unitTestPassPercentage = "${config.unitTestCases.passPercentage}"
	
	ctestJunitXSLPath = "${config.unitTestCases.ctestJunitXSLPath}"
	skipSonarAnalysis = "${config.sonar.skipSonarAnalysis}"
	sonarProjectAuthToken = "${config.sonar.projectAuthToken}"
	sonarProjectKey = "${config.sonar.projectKey}"
	qgSleepTime = "${config.sonar.sleepTime}"
	
	skipZephyrReports = "${config.zephyr.skipZephyrReports}"
	zephyrServerAddress = "${config.zephyr.serverAddress}"
	zephyrProjectKey = "${config.zephyr.projectKey}"
	zephyrReleaseKey = "${config.zephyr.releaseKey}"
	zephyrParserTemplateKey = "${config.zephyr.parserTemplateKey}"
	zephyrCycleDuration = "${config.zephyr.cycleDuration}"
	zephyrCycleKey = "${config.zephyr.cycleKey}"
	zephyrCyclePrefix = "${config.zephyr.cyclePrefix}"
	zephyrCreatePackage = "${config.zephyr.createPackage}"
	
	skipTrivyScan = "${config.trivy.skipTrivyScan}"
	trivyCleanImgPath = "${config.trivy.trivyCleanImgPath}"
	trivyCleanImgName = "${config.trivy.trivyCleanImgName}"
	trivyCleanImgVersion = "${config.trivy.trivyCleanImgVersion}"
	severity = "${config.trivy.severity}"
	
	skipAccuricsReport = "${config.accurics.skipAccuricsReport}"
	accuricsConfigFilesLocation = "${config.accurics.configFilesLocation}"
	helmLocation = "${config.accurics.helmLocation}"
	terraformLocation = "${config.accurics.terrafomLocation}"
	
	skipHelmPush = "${config.helm.skipHelmPush}"
	helmChartLocation = "${config.helm.chartLocation}"
	
	harborURL = "${config.harbor.URL}"
	harborCredID = "${config.harbor.credID}"
	harborRepoName = "${config.harbor.repoName}"
	harborTagURL = "${config.harbor.tagURL}"
	
	skipImagePush = "${config.docker.skipImagePush}"
	dockerFileLocation = "${config.docker.dockerFileLocation}"
	imageName = "${config.docker.imageName}"
	tagName = "${config.docker.tagName}"
	
	skipCodeMerge = "${config.codeMerge.skipCodeMerge}"
	mergeToBranch = "${config.codeMerge.mergeToBranch}"
	
	emailToList = "${config.emailIDs.toList}" 
	emailCCList = "${config.emailIDs.ccList}" 
}

stages {
	stage('Build-Application') {
		steps {
			dir("${workspace}"){
				script {
					config =  loadValuesJSON()
					reportsPath = config.projectDetails.reportsPath
					sh "mkdir ${workspace}/${reportsPath} -p "
					if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("maven")) {
						def maven_Home = tool 'maven';
						sh "${maven_Home}/bin/mvn clean install -DskipTests"
					}
					if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("gradle")) {
						def gradle_Home = tool 'gradle';
						sh "${gradle_Home}/bin/gradle clean build"
					}
					if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("ant")) {
						sh 'echo runs Ant target compile using the build.xml file in the current directory.'
						def ant_Home = tool 'ant';
						sh "${ant_Home}/bin/ant -f ${antBuildFile} ${antCompileTarget}"
					}
					if ("${projectLanguage}".equalsIgnoreCase("Python")) {
						sh 'sudo apt install python3-pip'
						sh 'pip3 install -r requirements.txt'
					}
					if ("${projectLanguage}".equalsIgnoreCase("DotNet")) {
						sh 'dotnet clean'
						sh 'dotnet build'
					}
					if ("${projectLanguage}".equalsIgnoreCase("C") || "${projectLanguage}".equalsIgnoreCase("CPP") ) {
						sh "rm -rf ${cmakeBuildDir}"
						cmakeBuild buildDir: "${cmakeBuildDir}", buildType: "${cmakeBuildType}", generator: "${cmakeGenerator}", installation: "${cmakeInstallation}"
						sh '''
							cd '''+ cmakeBuildDir +'''
							make
						'''    
					}
				}
			}
		}
    }
	stage('Unit-TestCases') {
		when {
			expression {"${skipUnitTestCases}" != 'true'}
		}
		steps {
			dir("${workspace}"){
				script {
				    runUnitTestCases ()
				}
			}
		}
    }
	stage('Zephyr-Reports') {
		when {
			expression {"${skipZephyrReports}" != 'true' && "${skipUnitTestCases}" != 'true'}
		}
		steps {
			dir("${workspace}"){
				script {
					zeeReporter createPackage: "${zephyrCreatePackage}", cycleDuration: "${zephyrCycleDuration}", cycleKey: "${zephyrCycleKey}", cyclePrefix: '', parserTemplateKey: "${zephyrParserTemplateKey}", projectKey: "${zephyrProjectKey}", releaseKey: "${zephyrReleaseKey}", resultXmlFilePath: "${jUnitTestResultsPath}/*.xml", serverAddress: "${zephyrServerAddress}"
					//zeeReporter createPackage: false, cycleDuration: '30 days', cycleKey: '2', cyclePrefix: '', parserTemplateKey: '1', projectKey: '1', releaseKey: '1', resultXmlFilePath: '', serverAddress: 'https://unisys.zephyrdemo.com'
				}
			}
		}
	}
	stage('Create Jira Issues for Failed Unit Test Cases') {
		when {
			expression {"${skipUnitTestCases}" != 'true'}
		}
		steps {
			dir("${workspace}"){
				script {
    				sh "echo UnitTestReportsPath: ${jUnitTestResultsPath}/*.xml"
					junit (
    					testResults: "${jUnitTestResultsPath}/*.xml",
    					skipPublishingChecks: "${skipPublishingChecks}",
    					testDataPublishers: [
    						jiraTestResultReporter(
    							configs: [
    								jiraStringField(fieldKey: 'summary', value: '${DEFAULT_SUMMARY}'),
    								jiraStringField(fieldKey: 'description', value: '${DEFAULT_DESCRIPTION}'),
    								jiraStringArrayField(fieldKey: 'labels', values: [jiraArrayEntry(value: 'Jenkins'), jiraArrayEntry(value:'Integration')])
    							],
    							projectKey: "${jiraProjectKey}",
    							issueType: "${jiraIssueType}",
    							autoRaiseIssue: "${jiraAutoRaiseIssue}",
    							autoResolveIssue: "${jiraAutoResolveIssue}",
    							autoUnlinkIssue: "${jiraAutoUnlinkIssue}",
    						)
    					]
                    )
				    def unitTestResults = junit (testResults: "${jUnitTestResultsPath}/*.xml", skipPublishingChecks: "${skipPublishingChecks}")
				    def totalUnitTestCases =  "${unitTestResults.totalCount}".toInteger()
				    def passedUnitTestCases =  "${unitTestResults.passCount}".toInteger()
				    def unitTestPassPercentage = "${unitTestPassPercentage}".toInteger() 
				    def passPercentage = 100 - (((totalUnitTestCases-passedUnitTestCases)/totalUnitTestCases)*100)
				    echo "passPercentage: ${passPercentage}"
				    if (passPercentage < unitTestPassPercentage) {
				        echo "unitTestPassPercentage: ${passPercentage}" 
				        currentBuild.result = "ABORTED"
                        throw new Exception("Percentage of passed test cases is less than ${unitTestPassPercentage}")
				    }
    			}    
			}
		}
	}
		
 	stage('SonarAnalysis-And-QualityGate') {
		when {
			expression {"${skipSonarAnalysis}" != 'true'}
		}
		steps {
			script {
				dir("${workspace}"){
					withSonarQubeEnv('sonarqube'){
						withCredentials([string(credentialsId: "${sonarProjectAuthToken}", variable: 'sonarUsrToken')]) {
							if ("${projectLanguage}".equalsIgnoreCase("Java") || "${projectLanguage}".equalsIgnoreCase("Python")) {
								def scanner_Home = tool 'sonarqube-scanner';
								sh "${scanner_Home}/bin/sonar-scanner -Dsonar.login=${sonarUsrToken} -Dsonar.projectKey=${sonarProjectKey} -Dsonar.java.binaries=."
								sh "echo Please refer sonarscanner usage for any clarifications: https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/"
							
							}
							if ("${projectLanguage}".equalsIgnoreCase("DotNet") && "${dotNetAppType}".equalsIgnoreCase("DotNetCore")) {
								def scanner_Home = tool 'sonar-scanner-msbuild';
								sh "below steps for .net core sonar scanner"
								sh "dotnet ${scanner_Home}/SonarScanner.MSBuild.dll begin /k:${sonarProjectKey} /d:sonar.login=${sonarUsrToken}"
								sh 'dotnet build ${dotNetSolutionPath}'
								sh "dotnet ${scanner_Home}/SonarScanner.MSBuild.dll end /d:sonar.login=${sonarUsrToken}"
								sh "echo Please refer sonarscannermsbuild usage for any clarifications: https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-msbuild/"
							}
							if ("${projectLanguage}".equalsIgnoreCase("C") || "${projectLanguage}".equalsIgnoreCase("CPP")) {
								def scanner_Home = tool 'sonarqube-scanner';
								sh "${scanner_Home}/bin/sonar-scanner -Dsonar.login=${sonarUsrToken} -Dsonar.projectKey=${sonarProjectKey}"
							}
						}	
					}
					//Get QualityGate status:
					sleep(time: "${qgSleepTime}", unit: 'MINUTES')
					def qg = waitForQualityGate()
					if (qg.status != 'OK'){
						sh 'echo If Pipeline fails due to "PENDING" or "INPROGRESS" please increase the time based on the report upload time.'
						error "Pipeline aborted due to quality gate failure: ${qg.status}"
					}
				}
			}
		}
	}
	stage('Docker-Build') {
		steps {
			dir("${workspace}"){
				script {
					sh "docker build -f $workspace/${dockerFileLocation} -t ${imageName} ."
				}
			}	
		}
    }
	stage('Trivy-Scan') {
		when {
			expression {"${skipTrivyScan}" != 'true' }
		}
		steps {
			dir("${workspace}"){
				script {
					sh "echo scanning application image with Trivy" 
					sh "docker load --input ${trivyCleanImgPath}"
					sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock ${trivyCleanImgName}:${trivyVersion} -q image --exit-code 1 --no-progress -s '${severity}' ${imageName}"
				}
			}
		}
    }
	stage('Image-Push-To-Harbor') {
		when {
			expression {"${skipImagePush}" != 'true' }
		}
		steps {
			dir("${workspace}"){
				withDockerRegistry([ url: "${harborURL}", credentialsId:"${harborCredID}"]) {
					script{
						sh 'echo harbor certificate need to be placed under /usr/local/share/ca-certificates in Jenkins node'  
						sh 'docker tag ${imageName}:latest ${harborTagURL}/${harborRepoName}/${imageName}:$BUILD_NUMBER'
						sh 'docker tag ${imageName}:latest ${harborTagURL}/${harborRepoName}/${imageName}:latest'
						sh 'docker push ${harborTagURL}/${harborRepoName}/${imageName}:$BUILD_NUMBER'
						sh 'docker push ${harborTagURL}/${harborRepoName}/${imageName}:latest'
					}
				}
			}
		}
	}
	stage('Accurics-Reports') {
		when {
			expression {"${skipAccuricsReport}" != 'true'}
		}
		steps {
			dir("${workspace}"){
				script {
					sh "sudo cp ${workspace}/${accuricsConfigFilesLocation}/accurics /usr/local/bin/"
					sh 'chmod +x /usr/local/bin/accurics'
					sh 'export PATH=$PATH:/usr/local/bin/'
					if (env.terraformLocation != ' ') {
						dir("${workspace}/${terraformLocation}") {
							sh "sudo rm -rf config"
							sh "sudo cp -f ${workspace}/${accuricsConfigFilesLocation}/config-terraform ${workspace}/${terraformLocation}/config"
							sh "accurics init"
							sh "accurics plan"
							sh "sudo cp accurics_report.html ${workspace}/${reportsPath}/reportTerraform.html"
						}
					}
					if (env.helmLocation != ' ') {
						dir("${workspace}/${helmLocation}") {
							sh "sudo rm -rf config"
							sh "cp ${workspace}/${accuricsConfigFilesLocation}/config-helm ${workspace}/${helmLocation}/config"
							sh "accurics scan helm"
							sh "sudo cp accurics_report.html ${workspace}/${reportsPath}/reportHelm.html"
							sh "sudo rm -rf config"
							sh "cp ${workspace}/${accuricsConfigFilesLocation}/config-kubernetes ${workspace}/${helmLocation}/config"
							sh "accurics scan k8s"
							sh "sudo cp accurics_report.html ${workspace}/${reportsPath}/reportk8s.html"
						}
					}	
				}
			}
		}
    }
	stage('Helm-Push-To-Harbor') {
		when {
			expression {"${skipHelmPush}" != 'true'}
		}
		steps {
			dir("${workspace}/${helmChartLocation}"){
				script {
					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${harborCredID}", usernameVariable: 'username', passwordVariable: 'password']]) {
						sh('''
						helm repo add ${harborRepoName} ${harborURL}/chartrepo/${harborRepoName} --username $username --password $password --force-update
						helm repo update
                        rm -f *.tgz
                        helm package .
						helm cm-push *.tgz ${harborRepoName}
						''')
					}
				}
			}	
		}
	}
	stage('Merge-Code-To-Release-Branch') {
	    when {
			expression {"${skipCodeMerge}" != 'true'}
		}
		steps {
			dir("${workspace}"){
				script {
					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: "${GitCred}", usernameVariable: 'username', passwordVariable: 'password']]) {	
						sh('''
						git config --local user.name ${username}
						git config --local user.password ${password}
						git config --global credential.helper store
						git checkout master
						git push origin master:"${mergeToBranch}"
						''') 
					}
				}
			}	
		}
	}
}

post {
	aborted {
		script{
			def createJiraIssue = [fields: [ project: [key: "${jiraProjectKey}"], issuetype: [id: "${jiraIssuetype}"], summary: "ABORTED build ${currentBuild.displayName} for the  JOB ${env.JOB_NAME}", description: "Build log is here: ${env.BUILD_URL}/console "]]
			response = jiraNewIssue issue: createJiraIssue, site: "${jiraSite}"
		}		
    }
    failure {
		script{
			sh "echo Creating Jira ticket for FAILED  Build"
			def createJiraIssue = [fields: [ project: [key: "${jiraProjectKey}"], issuetype: [id: "${jiraIssuetype}"], summary: "FAILED build ${currentBuild.displayName} for the  JOB ${env.JOB_NAME}", description: "Build log is here: ${env.BUILD_URL}/console "]]
			response = jiraNewIssue issue: createJiraIssue, site: "${jiraSite}"
		}		
    }
	always{
		script{
			sh "mkdir /tmp/${reportsPath}/ -p"
			sh "mkdir /tmp/${reportsPath}/${env.JOB_NAME} -p"
			sh "mkdir /tmp/${reportsPath}/${env.JOB_NAME}/${env.BUILD_NUMBER} -p"
			sh "cp ${workspace}/${reportsPath}/  /tmp/${reportsPath}/${env.JOB_NAME}/${env.BUILD_NUMBER} -r"
			echo "This build Reports are available on the Location: /tmp/${reportsPath}/${env.JOB_NAME}/${env.BUILD_NUMBER}"
			
			emailext (
				to: "${emailToList}",
				subject: "JOB ${env.JOB_NAME} , BUILD ${currentBuild.displayName} completed with status: ${currentBuild.currentResult}",
				body: '''${SCRIPT, template="groovy-html.template"}''',
				mimeType: 'text/html',
				attachLog: true,
				attachmentsPattern: "${reportsPath}/*"
			)
			cleanWs notFailBuild: true
		}	
	}
}

}
def loadValuesJSON(){
    def valuesJson = readJSON file: 'config.json'
	return valuesJson;
}

def runUnitTestCases() {
    config =  loadValuesJSON()
	projectLanguage = config.projectDetails.language
	buildTool = config.build.buildTool
	skipPublishingChecks = config.unitTestCases.skipPublishingChecks 
	jUnitTestResultsPath = config.unitTestCases.jUnitTestResultsPath 
	pythonUnitTestFileLocation = config.unitTestCases.pythonUnitTestFileLocation 
	dotNetUnitTestSrcPath = config.unitTestCases.dotNetUnitTestSrcPath 
	unitTestPassPercentage = config.unitTestCases.passPercentage 
	reportsPath = config.projectDetails.reportsPath
	
    if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("maven")) {
		def maven_Home = tool 'maven';
		sh "${maven_Home}/bin/mvn test -B -Dmaven.test.failure.ignore=true clean verify"
		sh "${maven_Home}/bin/mvn surefire-report:report"
		sh "sudo cp ${workspace}/target/site/surefire-report.html ${workspace}/${reportsPath}/reportUnitTest.html"
	}
	if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("gradle")) {
		def gradle_Home = tool 'gradle';
		sh "${gradle_Home}/bin//gradle clean test --no-daemon"
   	}
	if ("${projectLanguage}".equalsIgnoreCase("Java") && "${buildTool}".equalsIgnoreCase("ant")) {
		sh 'echo runs Ant target unittest using the build.xml file in the current directory.'
		def ant_Home = tool 'ant';
		sh "${ant_Home}/bin/ant -f ${antBuildFile} ${antUnitTestTarget}"
	}					
	if ("${projectLanguage}".equalsIgnoreCase("Python")) {
		try {
			ssh "pip3 install pytest-django"
			sh "py.test --junitxml=${jUnitTestResultsPath}/junitXML.xml"
		}catch (error) {
			sh "echo error: ${error}"
		}
	}
	if ("${projectLanguage}".equalsIgnoreCase("DotNet")) {
		try {
			sh('''
				mkdir ${jUnitTestResultsPath}
				cd ${dotNetUnitTestSrcPath}
				dotnet add package JUnitTestLogger --version 1.1.0
				dotnet test --logger:junit
			''')
		}catch (error) {
			sh "cp ${dotNetUnitTestSrcPath}/TestResults/*.xml ${workspace}/${jUnitTestResultsPath}/"
			sh "echo error: ${error}"
		}
	}
	if ("${projectLanguage}".equalsIgnoreCase("C") || "${projectLanguage}".equalsIgnoreCase("CPP") ) {
		try {
			dir("${workspace}/${cmakeBuildDir}") {
			    ctest installation: 'InSearchPath',  arguments: '-T Test'
			    sh '''
    				sudo apt-get install -y xsltproc
    				if [ ! -f Testing/TAG ]; then
    					echo "No Testing/TAG file found, did ctest successfully run?" 1>&2
    					exit 1
    				fi
    				latest_test=`head -n 1 < Testing/TAG`
    				chmod +x Testing/$latest_test/Test.xml
    				results=Testing/$latest_test/Test.xml
    				xsltproc ../'''+ ctestJunitXSLPath +''' $results > junitXML.xml
			    '''
			}    
			
		} catch (e) {
		    dir("${workspace}/${cmakeBuildDir}") {
    			sh '''
    				if [ ! -f Testing/TAG ]; then
    					echo "No Testing/TAG file found, did ctest successfully run?" 1>&2
    					exit 1
    				fi
    				latest_test=`head -n 1 < Testing/TAG`
    				chmod +x Testing/$latest_test/Test.xml
    				results=Testing/$latest_test/Test.xml
    				xsltproc ../'''+ ctestJunitXSLPath +''' $results > junitXML.xml
    			'''
		    }
		}
	}

}

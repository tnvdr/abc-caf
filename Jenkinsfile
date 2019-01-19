pipeline {
  agent any
  
  /*parameters {
	string(
		name: 'CICD Pipeline',
		defaultValue:'WB',
		description: 'Costa CICD Pipeline')
    }*/
	stages {
    stage('Pipeline Start') {
      steps {
       echo 'Pipeline Started'
      }
    }
}
}
node {
	properties([
     parameters([
       choiceParam(
         choices: 'DEV\nTEST\nSTAGE\nPROD',
            description: 'Please selet the Build ENVIRONMENTS',
            name: 'ENVIRONMENTS'
       ),
       choiceParam(
         choices: 'BUILD\nBUILD-AND-ARTEFACT-UPLOAD\nBUILD-AND-RELEASE\nRELEASE',
            description: 'Please select the Build Mechanism',
            name: 'BUILD_MECHANISM'
       ),
	   string(
	   name: 'BuildParameters', 
	   defaultValue: 'mvn -U install -DskipTests=true', 
	   description: 'Please provide Additional Build Parameter to execute')
     ])
   ])	
   
	if(params.BUILD_MECHANISM == 'BUILD'){
		stage 'SourceCodeBuild'	
			UDF_BuildSourceCode()
			
		stage 'SonarQube'
			UDF_ExecuteSonarQubeRules()
	}	
	else if(params.BUILD_MECHANISM == 'BUILD-AND-RELEASE'){
		try{
		stage 'SourceCodeBuild'	
			UDF_BuildSourceCode()
			
		stage 'SonarQube'
			UDF_ExecuteSonarQubeRules()
		
		/*def EnvironmentNameUserInput = input(
			 id: 'EnvironmentNameUserInput', message: 'Enter Environment specific Profile name for CloudHub Deployment:?', 
			 parameters: [
			 [$class: 'TextParameterDefinition', defaultValue: '', description: 'CloudHub Environment Name', name: 'EnvironmentName']
			])*/
			
		def DomainNameUserInput = input(
			 id: 'DomainNameUserInput', message: 'Enter app/domain name for CloudHub Deployment:?', 
			 parameters: [
			 [$class: 'TextParameterDefinition', defaultValue: '', description: 'CloudHub Domain Name', name: 'DomainName']
			])
		
		def nexus_Protocol = "http"
		def nexus_BaseURL = "${env.LOCAL_NEXUS_BASEURL}"		
		def nexus_RepoName = UDF_Get_Nexus_RepoName("${params.ENVIRONMENTS}")		
		def pom_GroupID = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","groupId")
		def pom_ArtifactId = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","artifactId")
		def pom_Version = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","version")		
		//def nexus_SearchURL = "${nexus_BaseURL}/service/rest/beta/search?repository=${nexus_RepoName}&group=${pom_GroupID}&name=${pom_ArtifactId}"
		
		def propertiesFilePath = "${env.JENKINS_HOME}/CloudHub/"+UDF_GetGitRepoName()+"/${params.ENVIRONMENTS}.properties.txt"
		def downloadDir = "${env.JENKINS_HOME}/CloudHub/Downloads/"+UDF_GetGitRepoName()	
		//def downloadFilePath="${env.JENKINS_HOME}/CloudHub/Downloads/"+UDF_GetGitRepoName()+"/${pom_ArtifactId}.zip"
		def downloadFilePath="${env.WORKSPACE}/target/${pom_ArtifactId}-${pom_Version}.zip"
	
		stage 'ArtifactUploadToNexus'
			UDF_ArtifactUploadToNexus(nexus_BaseURL,pom_GroupID,pom_Version,nexus_RepoName,pom_ArtifactId,nexus_Protocol)
			
		stage 'DeployToCloudHub'
			UDF_DeployToCloudHub(downloadFilePath, propertiesFilePath,"",DomainNameUserInput)
		
		stage 'Notification'
			SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","success")
			
		}catch(error)
		{
			throw(error)
			SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","Failed")
		}
	
	}
	else if(params.BUILD_MECHANISM == 'BUILD-AND-ARTEFACT-UPLOAD'){
		try{
		stage 'SourceCodeBuild'	
			UDF_BuildSourceCode()
			
		stage 'SonarQube'
			UDF_ExecuteSonarQubeRules()
		
		def nexus_Protocol = "http"
		def nexus_BaseURL = "${env.LOCAL_NEXUS_BASEURL}"		
		def nexus_RepoName = UDF_Get_Nexus_RepoName("${params.ENVIRONMENTS}")		
		def pom_GroupID = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","groupId")
		def pom_ArtifactId = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","artifactId")
		def pom_Version = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","version")		
		//def nexus_SearchURL = "${nexus_BaseURL}/service/rest/beta/search?repository=${nexus_RepoName}&group=${pom_GroupID}&name=${pom_ArtifactId}"
		
		def propertiesFilePath = "${env.JENKINS_HOME}/CloudHub/"+UDF_GetGitRepoName()+"/${params.ENVIRONMENTS}.properties.txt"
		def downloadDir = "${env.JENKINS_HOME}/CloudHub/Downloads/"+UDF_GetGitRepoName()	
		//def downloadFilePath="${env.JENKINS_HOME}/CloudHub/Downloads/"+UDF_GetGitRepoName()+"/${pom_ArtifactId}.zip"
		//def downloadFilePath="${env.WORKSPACE}/target/${pom_ArtifactId}-${pom_Version}.zip"
	
		stage 'ArtifactUploadToNexus'
			UDF_ArtifactUploadToNexus(nexus_BaseURL,pom_GroupID,pom_Version,nexus_RepoName,pom_ArtifactId,nexus_Protocol)
			
		}catch(error)
		{
			throw(error)
			SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","Failed")
		}
	
	}
	if(params.BUILD_MECHANISM == 'RELEASE'){
		try{
		
		stage 'GetArtifactListFromNexus'			
		
			def nexus_Protocol = "http"
			def nexus_BaseURL = "${env.LOCAL_NEXUS_BASEURL}"		
			def nexus_RepoName = UDF_Get_Nexus_RepoName("${params.ENVIRONMENTS}")		
			def pom_GroupID = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","groupId")
			def pom_ArtifactId = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","artifactId")
			def pom_Version = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","version")		
			//def nexus_SearchURL = "${nexus_BaseURL}/service/rest/beta/search?repository=${nexus_RepoName}&group=${pom_GroupID}&name=${pom_ArtifactId}"
			def downloadDir = ""
			environment {
			NEXUS_CREDENTIALS = credentials('cf5cf9a9-4ceb-4372-abb3-31e595c22bda')
			}
			withCredentials([usernamePassword(
			credentialsId: 'cf5cf9a9-4ceb-4372-abb3-31e595c22bda',
			passwordVariable: 'nexuspassword',
			usernameVariable: 'nexususername')])
			{
			def nexus_SearchURL = "curl -v -u ${nexususername}:${nexuspassword} ${nexus_BaseURL}/service/rest/beta/search?repository=${nexus_RepoName}&group=${pom_GroupID}&name=${pom_ArtifactId}"
			def nexusVersionInput = input(
                id: 'nexusVersionInput', message: 'Please select Nexus Artifact Version for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: UDF_GetNexusArtifactsList(nexus_SearchURL), 
					name: 'SELECTED_NEXUS_VERSION',
					description: 'A select box option'
				]])
			String selectedVersion = "${nexusVersionInput}"
			selectedVersion = selectedVersion.split(":")[1]	
			
			def nexus_SearchURL_File = "curl -v -u ${nexususername}:${nexuspassword} ${nexus_BaseURL}/service/rest/beta/search?repository=${nexus_RepoName}&group=${pom_GroupID}&name=${pom_ArtifactId}&version=${selectedVersion}"
			downloadDir = UDF_GetNexusArtifacts_DownloadURL(nexus_SearchURL_File)
			}
			

			def DomainNameUserInput = input(
			 id: 'DomainNameUserInput', message: 'Enter app/domain name for CloudHub Deployment:?', 
			 parameters: [
			 [$class: 'TextParameterDefinition', defaultValue: '', description: 'CloudHub Domain Name', name: 'DomainName']
			])
			
			/*def EnvironmentNameUserInput = input(
			 id: 'EnvironmentNameUserInput', message: 'Enter Environment specific Profile name for CloudHub Deployment:?', 
			 parameters: [
			 [$class: 'TextParameterDefinition', defaultValue: '', description: 'CloudHub Environment Name', name: 'EnvironmentName']
			])*/
			
			def propertiesFilePath = "${env.JENKINS_HOME}/CloudHub/"+UDF_GetGitRepoName()+"/${params.ENVIRONMENTS}.properties.txt"
			//def downloadDir = "${env.JENKINS_HOME}/CloudHub/Downloads/"+UDF_GetGitRepoName()		
			
			/*withCredentials([usernamePassword(
			credentialsId: 'cf5cf9a9-4ceb-4372-abb3-31e595c22bda',
			passwordVariable: 'nexuspassword',
			usernameVariable: 'nexususername')])
			{
			def nexus_SearchURL_File = "curl -v -u ${nexususername}:${nexuspassword} ${nexus_BaseURL}/service/rest/beta/search?repository=${nexus_RepoName}&group=${pom_GroupID}&name=${pom_ArtifactId}&version=${selectedVersion}"
			def downloadDir = UDF_GetNexusArtifacts_DownloadURL(nexus_SearchURL_File)
			}*/
			def downloadFilePath="${env.JENKINS_HOME}/CloudHub/Downloads/"+UDF_GetGitRepoName()+"/${pom_ArtifactId}.zip"
			//def downloadFilePath="${env.WORKSPACE}/target/${pom_ArtifactId}-${pom_Version}.zip"

		stage 'DeployToCloudHub'
				UDF_DeployToCloudHub(downloadFilePath, propertiesFilePath,downloadDir,DomainNameUserInput)
			
		stage 'Notification'
			SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","success")
		
		}catch(error)
		{
			//SendEmail()
			throw(error)
			SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","Failed")
		}
	}
}

//BUILD STAGE
def UDF_BuildSourceCode()
{
	try	{
		if (params.BuildParameters == '')
		{
	echo 'Build is Starting'
	sh 'mvn -U install -DskipTests=true'
	echo 'Build Completed'	
		}else
		{
		echo 'Build is Starting'
		sh '${BuildParameters}'
		echo 'Build Completed'	
		}
	}catch(error)
	{
		throw(error)
		SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","Failed")
	}
}

//SONARQUBE - STAGE
def UDF_ExecuteSonarQubeRules()
{
	try{
	echo 'SonarQube Rules Execution started'
	withSonarQubeEnv('SonarServer-Local') {
		sh 'mvn sonar:sonar'
	}
	echo 'SonarQube Rules Execution Completed'
	}catch(error)
	{
		throw(error)
		SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","Failed")
	}
}

//NEXUS ARTIFACT UPLOAD - STAGE
def UDF_ArtifactUploadToNexus(udfp_NexusBaseURL, udfp_GroupId, udfp_Version, udfp_NexusRepoName, udfp_ArtifactId, udfp_Protocol)
{
	try{
	echo 'Artifact Copy to Nexus Started'
	
	String nexusRepoName = "${udfp_NexusRepoName}/"
	String targetZipName = "target/${udfp_ArtifactId}-${udfp_Version}.zip"
	
	if(udfp_NexusBaseURL.contains("http://"))
	{
		udfp_NexusBaseURL = udfp_NexusBaseURL.substring(7)
	}
	else if(udfp_NexusBaseURL.contains("https://"))
	{
		udfp_NexusBaseURL = udfp_NexusBaseURL.substring(8)
	}
	
	
	nexusArtifactUploader(
		nexusVersion: 'nexus3',
		protocol: udfp_Protocol,
		nexusUrl: udfp_NexusBaseURL,
		groupId: udfp_GroupId,
		version: udfp_Version,
		repository: nexusRepoName,
		credentialsId: '5ddd1595-a2f5-4bc3-8f57-4ddd2d14ab2c',
		artifacts: [
			[artifactId: udfp_ArtifactId,
			 classifier: '',
			 file: targetZipName,
			 type: 'zip']
		]
	 )
	 
    echo 'Artifact Copy to Nexus Completed'
	}catch(error)
	{
		throw(error)
		SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","Failed")
	}
}

/*
DEPLOY STAGE
This function provides functionality to deploy the applicatin package(zip file) to CloudHub Runtime
*/
def UDF_DeployToCloudHub(udfp_DownloadedFilePath, udfp_PropertiesFilePath, udfp_NexusPackageURL, udfp_AppName)
{
    /*
	Download the selected Application Zip package from Nexus Repository
	*/
	
	def AnypointCredentialID = input(
			 id: 'DomainNameUserInput', message: 'Please provide Cloudhub Credential ID:?', 
			 parameters: [
			 [$class: 'TextParameterDefinition', defaultValue: '', description: 'Cloudhub Credential', name: 'DomainName']
			])
			
	def PropertiesFileInput = input(
                id: 'PropertiesFileInput', message: 'Is Properties file required for the deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: 'YES\nNO', 
					name: 'SELECTED_PROPERTY_FILE',
					description: 'Please confirm YES or NO'
				]])
		
	def AnypointOrganization = input(
                id: 'AnypointOrganization', message: 'Please select CloudHub Anypoint Organization name to deploy', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: 'Costa Coffee\nWhitbread Group\nWhitbread', 
					name: 'ANYPOINT_ORGANIZATION',
					description: 'Please select the Organization'
				]])
	def AnypointEnvironment = input(
			 id: 'AnypointEnvironment', message: 'Please provide Cloudhub Anypoint Environment name to deploy', 
			 parameters: [
			 [$class: 'TextParameterDefinition', defaultValue: 'Example: CC-ELK-DEV', description: 'Cloudhub Environment Name', name: 'DomainName']
			])
		
	environment {
        ANYPOINT_CREDENTIALS = credentials("${AnypointCredentialID}")
		}
	
	echo "Nexus Download URL - ${udfp_NexusPackageURL}"
	
	if(udfp_NexusPackageURL != "")
	{
		withCredentials([usernamePassword(
			credentialsId: 'cf5cf9a9-4ceb-4372-abb3-31e595c22bda',
			passwordVariable: 'nexuspassword',
			usernameVariable: 'nexususername')])
			{
		sh "wget --user ${nexususername} --password ${nexuspassword} ${udfp_NexusPackageURL} -O ${udfp_DownloadedFilePath}"	
			}
	}
	
	def appExists = false
	try{		
		echo 'Auto-deploy to CloudHub started'
		withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
		sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application describe ${udfp_AppName}
		"""
			}
		appExists = true
	}catch(err)
	{
		//throw(error)
		SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","FAILED")
	}
	
	if(appExists == false && PropertiesFileInput == 'YES')
	{
		def workerSizeInput = input(
                id: 'workerSizeInput', message: 'Please select vCores for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '0.1\n0.2\n1\n2\n4\n8\n16', 
					name: 'SELECTED_WORKER_VERSION',
					description: 'Please select worker version'
				]])
		def workerNumberInput = input(
                id: 'workerInput', message: 'Please select workerSize for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '1\n1\n3\n4\n5\n6\n7\n8', 
					name: 'SELECTED_WORKERS',
					description: 'Please select No.of workers'
				]])
		def RuntimeInput = input(
                id: 'workerInput', message: 'Please select mule runtime for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '3.8.0\n3.8.1\n3.8.2\n3.8.3\n3.8.4\n3.8.5\n3.8.6\n3.9.0', 
					name: 'SELECTED_MULE_RUNTIME',
					description: 'Please select Mule Runtime'
				]])
				
		vCoreInput = "${workerSizeInput}"
		workerInput = "${workerNumberInput}"
		runTimeVersion = "${RuntimeInput}"
				
		
		//Deploy New Application
		withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
			sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application deploy ${udfp_AppName} ${udfp_DownloadedFilePath} --workerSize ${vCoreInput} --workers ${workerInput} --runtime ${runTimeVersion} --propertiesFile ${udfp_PropertiesFilePath}
		"""
			}
		echo 'new app and props'
	}
	else if(appExists == false && PropertiesFileInput == 'NO')
	{
		//Modify Existing Application
		def workerSizeInput = input(
                id: 'workerSizeInput', message: 'Please select vCores for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '0.1\n0.2\n1\n2\n4\n8\n16', 
					name: 'SELECTED_WORKER_VERSION',
					description: 'Please select worker version'
				]])
		def workerNumberInput = input(
                id: 'workerInput', message: 'Please select workerSize for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '1\n1\n3\n4\n5\n6\n7\n8', 
					name: 'SELECTED_WORKERS',
					description: 'Please select No.of workers'
				]])
		def RuntimeInput = input(
                id: 'workerInput', message: 'Please select mule runtime for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '3.8.0\n3.8.1\n3.8.2\n3.8.3\n3.8.4\n3.8.5\n3.8.6\n3.9.0', 
					name: 'SELECTED_MULE_RUNTIME',
					description: 'Please select Mule Runtime'
				]])
				
		vCoreInput = "${workerSizeInput}"
		workerInput = "${workerNumberInput}"
		runTimeVersion = "${RuntimeInput}"
		
		withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
			sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application deploy ${udfp_AppName} ${udfp_DownloadedFilePath} --workerSize ${vCoreInput} --workers ${workerInput} --runtime ${runTimeVersion}
		"""
			}
		echo 'new app and no props'
	}
	else if (appExists == true && PropertiesFileInput == 'YES')
	{
		//Modify Existing Application
		def MuleRunTImeProperties = input(
                id: 'MuleRunTImeProperties', message: 'Do you want to change Mule Runtime like vCore,Workers, MuleRuntimeVersion:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: 'YES\nNO', 
					name: 'MULE_RUNTIME_PROPERTIES',
					description: 'Please confirm YES or NO'
				]])
		if (MuleRunTImeProperties == 'YES')
		{
			def workerSizeInput = input(
                id: 'workerSizeInput', message: 'Please select vCores for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '0.1\n0.2\n1\n2\n4\n8\n16', 
					name: 'SELECTED_WORKER_VERSION',
					description: 'Please select worker version'
				]])
		def workerNumberInput = input(
                id: 'workerInput', message: 'Please select workerSize for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '1\n1\n3\n4\n5\n6\n7\n8', 
					name: 'SELECTED_WORKERS',
					description: 'Please select No.of workers'
				]])
		def RuntimeInput = input(
                id: 'workerInput', message: 'Please select mule runtime for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '3.8.0\n3.8.1\n3.8.2\n3.8.3\n3.8.4\n3.8.5\n3.8.6\n3.9.0', 
					name: 'SELECTED_MULE_RUNTIME',
					description: 'Please select Mule Runtime'
				]])
				
		vCoreInput = "${workerSizeInput}"
		workerInput = "${workerNumberInput}"
		runTimeVersion = "${RuntimeInput}"
		
		withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
			sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application modify ${udfp_AppName} ${udfp_DownloadedFilePath} --workerSize ${vCoreInput} --workers ${workerInput} --runtime ${runTimeVersion} --propertiesFile ${udfp_PropertiesFilePath}
		"""
			}
		echo 'existing app and props'
		}else{
			withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
			sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application modify ${udfp_AppName} ${udfp_DownloadedFilePath} --propertiesFile ${udfp_PropertiesFilePath}
		"""
			}
			
		}
	}
	else if (appExists == true && PropertiesFileInput == 'NO')
	{
		//Modify Existing Application
		
		def MuleRunTImeProperties = input(
                id: 'MuleRunTImeProperties', message: 'Do you want to change Mule Runtime like vCore,Workers, MuleRuntimeVersion:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: 'YES\nNO', 
					name: 'MULE_RUNTIME_PROPERTIES',
					description: 'Please confirm YES or NO'
				]])
				
		if (MuleRunTImeProperties == 'YES')
		{
			def workerSizeInput = input(
                id: 'workerSizeInput', message: 'Please select vCores for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '0.1\n0.2\n1\n2\n4\n8\n16', 
					name: 'SELECTED_WORKER_VERSION',
					description: 'Please select worker version'
				]])
		def workerNumberInput = input(
                id: 'workerInput', message: 'Please select workerSize for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '1\n1\n3\n4\n5\n6\n7\n8', 
					name: 'SELECTED_WORKERS',
					description: 'Please select No.of workers'
				]])
		def RuntimeInput = input(
                id: 'workerInput', message: 'Please select mule runtime for Deployment:?', 
                parameters: [
				[
					$class: 'ChoiceParameterDefinition', choices: '3.8.0\n3.8.1\n3.8.2\n3.8.3\n3.8.4\n3.8.5\n3.8.6\n3.9.0', 
					name: 'SELECTED_MULE_RUNTIME',
					description: 'Please select Mule Runtime'
				]])
				
		vCoreInput = "${workerSizeInput}"
		workerInput = "${workerNumberInput}"
		runTimeVersion = "${RuntimeInput}"
		/*environment {
        ANYPOINT_CREDENTIALS = credentials("${AnypointCredentialID}")
		}*/
		echo '${AnypointOrganization}'
		withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
		
		sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application modify ${udfp_AppName} ${udfp_DownloadedFilePath} --workerSize ${vCoreInput} --workers ${workerInput} --runtime ${runTimeVersion} --propertiesFile ${udfp_PropertiesFilePath}
		"""
			}
		echo 'existing app and props'
		}
		else{
		withCredentials([usernamePassword(
			credentialsId: "${AnypointCredentialID}",
			passwordVariable: 'password1',
			usernameVariable: 'username1')])
			{
			sh """			
			export ANYPOINT_USERNAME=${username1}
			export ANYPOINT_PASSWORD=${password1}
			export ANYPOINT_ORG="${AnypointOrganization}"
			export ANYPOINT_ENV="${AnypointEnvironment}"
			anypoint-cli runtime-mgr cloudhub-application modify ${udfp_AppName} ${udfp_DownloadedFilePath}
		"""
			}
		}
	}
}

def UDF_GetGitRepoName()
{
	try{
	def tokens = "${env.JOB_NAME}".tokenize('/')
    String repo = tokens[tokens.size()-2]
	
    //branch = tokens[tokens.size()-1]	
    //org = tokens[tokens.size()-3]	
	
	return repo
	}catch(error)
	{
		throw(error)
		SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","Failed")
	}
}

def UDF_Get_Nexus_RepoName(udfp_Environment)
{	
	try{
	echo "Start of UDF_Get_Nexus_RepoName"
	def repoName = ""
	
	if(udfp_Environment == "DEV" || udfp_Environment == "TEST" || udfp_Environment == "STAGE")
	{		
		repoName = "${env.LOCAL_NEXUS_NONPROD_REPONAME}"
	}
	else if(udfp_Environment == "PROD")
	{		
		repoName = "${env.LOCAL_NEXUS_PROD_REPONAME}"
	}
	echo "End of UDF_Get_Nexus_RepoName"
	
	return repoName
	}catch(error)
	{
		throw(error)
		SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","Failed")
	}
}

def UDF_GetPOMData(udfp_PomName, udfp_PropertyName)
{
	try{
	def resultVal = ""
	def pomFile = readFile(udfp_PomName)
	def pom = new XmlParser().parseText(pomFile)
	def gavMap = [:]
	resultVal =  pom[udfp_PropertyName].text().trim()
	return resultVal
	}catch(error)
	{
		throw(error)
		SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","Failed")
	}
}

/*
This function returns the list of Packages from given Nexus Repository
*/
def UDF_GetNexusArtifactsList(udfp_NexusPackageSearchURL)
{
	try{
	String responseString = ""
	//String nexusResponseJson = udfp_NexusPackageSearchURL.toURL().text
	def nexusResponseJson = sh(returnStdout: true, script: udfp_NexusPackageSearchURL).trim()
	
	//def nexusResponseJson = sh udfp_NexusPackageSearchURL
	
	//echo "After function call - Output:${nexusResponseJson}"		
	
	def nexusResponseJsonMap = parseJsonToMap(nexusResponseJson)
	
	for (int i = 0; i < nexusResponseJsonMap.items.size(); i++) {
		responseString = responseString+ nexusResponseJsonMap.items[i].name+":"+nexusResponseJsonMap.items[i].version+"\n"
	}
	
	return responseString
	}catch(error)
	{
		throw(error)
		SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","Failed")
	}
}

def UDF_GetNexusArtifacts_DownloadURL(udfp_NexusPackageSearchURL)
{
	try{
	String responseString = ""
	//String nexusResponseJson = udfp_NexusPackageSearchURL.toURL().text
	
	def nexusResponseJson = sh(returnStdout: true, script: udfp_NexusPackageSearchURL).trim()
	
	echo "Function call - Output:${nexusResponseJson}"		
	
	def nexusResponseJsonMap = parseJsonToMap(nexusResponseJson)
	
	if(nexusResponseJsonMap.items.size() > 0)
	{			
		responseString = nexusResponseJsonMap.items[0].assets[0].downloadUrl
	}
	
	return responseString
	}catch(error)
	{
		throw(error)
		SendEmail("lp.admin@whishworks.com","lp.admin@whishworks.com","Failed")
	}
}

/*
This function Sends Email
*/
def SendEmail(udfp_ToAddress, udfp_FromAddress, udfp_Status)
{
	try{
	   String body = ""
	   
	   if(udfp_Status == "success")
	   {
		   body= "SUCCESS"
	   }
	   else
	   {
		   body= "FAILED"
	   }
		
		mail subject: "${env.JOB_NAME} (${env.BUILD_NUMBER}) ${body}",
				body: "It appears that ${env.BUILD_URL} is ${body}",
				  to: udfp_ToAddress,
			 replyTo: udfp_ToAddress,
				from: udfp_FromAddress 
				
	}catch(error)
	{		
		throw(error)
	}
}

/*
This function converts the given json string to HashMap and returns it back
*/
import groovy.json.JsonSlurperClassic
@NonCPS
def parseJsonToMap(String json) 
{
    final slurper = new JsonSlurperClassic()
    return new HashMap<>(slurper.parseText(json))
}

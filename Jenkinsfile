pipeline
{
    agent any
	tools
	{
		maven 'Maven'
	}
	options
    {
        // Append time stamp to the console output.
        timestamps()
      
        timeout(time: 1, unit: 'HOURS')
      
        // Do not automatically checkout the SCM on every stage. We stash what
        // we need to save time.
        skipDefaultCheckout()
      
        // Discard old builds after 10 days or 30 builds count.
        buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))
	  
	    //To avoid concurrent builds to avoid multiple checkouts
	    disableConcurrentBuilds()
    }
    stages
    {
	    stage ('checkout')
		{
			steps
			{
				checkout scm
			}
		}
		stage ('Build')
		{
			steps
			{
				bat "mvn install"
			}
		}
		stage ('Unit Testing')
		{
			steps
			{
				bat "mvn test"
			}
		}
		stage ('Sonar Analysis')
		{
			steps
			{
				withSonarQubeEnv("Test_Sonar") 
				{
					bat "mvn sonar:sonar"
				}
			}
		}
		stage ('Upload to Artifactory')
		{
			steps
			{
				rtMavenDeployer (
                    id: 'deployer',
                    serverId: '123456789@artifactory',
                    releaseRepo: 'sachinrana01',
                    snapshotRepo: 'sachinrana01'
                )
                rtMavenRun (
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: 'deployer',
                )
                rtPublishBuildInfo (
                    serverId: '123456789@artifactory',
                )
			}
		}
		stage ('Docker Image')
		{
			steps
			{
				bat returnStdout: true, script: 'docker build -t sachinrana01/java_sample_app:${BUILD_NUMBER} -f Dockerfile .'
			}
		}
		stage ('Push to DTR')
	    {
		    steps
		    {
		    	bat returnStdout: true, script: 'docker push sachinrana01/java_sample_app:${BUILD_NUMBER}'
		    }
	    }
        stage ('Stop Running container')
    	{
	        steps
	        {
	            bat '''
                    ContainerID=$(docker ps | grep 7016 | cut -d " " -f 1)
                    if [  $ContainerID ]
                    then
                        docker stop $ContainerID
                        docker rm -f $ContainerID
                    fi
                '''
	        }
	    }

		stage ('Docker deployment')
		{
		    steps
		    {
		        bat 'docker run --name devopssampleapplication_sachinrana01 -d -p 7016:8080 dtr.nagarro.com:443/devopssampleapplication_sachinrana:${BUILD_NUMBER}'
		    }
		}
	}
	post 
	{
        always 
		{
			emailext attachmentsPattern: 'report.html', body: '${JELLY_SCRIPT,template="health"}', mimeType: 'text/html', recipientProviders: [[$class: 'RequesterRecipientProvider']], replyTo: 'charu.garg@nagarro.com', subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: 'charu.garg@nagarro.com'
        }
    }
}

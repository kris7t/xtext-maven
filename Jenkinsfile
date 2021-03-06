// Tell Jenkins how to build projects from this repository
node {
	try {
		properties([
			[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', numToKeepStr: '15']]
		])
		
		stage 'Checkout'
		checkout scm
		
		dir('build') { deleteDir() }
		dir('.m2/repository/org/eclipse/xtext') { deleteDir() }
		dir('.m2/repository/org/eclipse/xtend') { deleteDir() }
		
		stage 'Maven Build'
		def workspace = pwd()
		def mvnHome = tool 'M3'
		env.M2_HOME = "${mvnHome}"
		try {
			sh "${mvnHome}/bin/mvn --batch-mode --update-snapshots -fae -PuseJenkinsSnapshots -Dmaven.test.failure.ignore=true -Dmaven.repo.local=${workspace}/.m2/repository clean deploy"
		} finally {
			step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/*.xml'])
		}
		archive 'build/**'
		
		if (currentBuild.result == 'UNSTABLE') {
			slackSend color: 'warning', message: "Build Unstable - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
		} else {
			slackSend color: 'good', message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
		}
	} catch (e) {
		slackSend color: 'danger', message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
		throw e
	}
}

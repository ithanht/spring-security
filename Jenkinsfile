def projectProperties = [
	[$class: 'BuildDiscarderProperty',
		strategy: [$class: 'LogRotator', numToKeepStr: '5']],
	pipelineTriggers([cron('@daily')])
]
properties(projectProperties)

parallel check: {
	stage('Check') {
		node {
			checkout scm
			try {
				sh "./gradlew clean check  --refresh-dependencies --no-daemon"
			} finally {
				junit '**/build/*-results/*/*.xml'
			}
		}
	}
},
sonar: {
	stage('Sonar') {
		node {
			checkout scm
			withCredentials([string(credentialsId: 'spring-sonar.login', variable: 'SONAR_LOGIN')]) {
				sh "./gradlew clean sonarqube -Dsonar.host.url=$SPRING_SONAR_HOST_URL -Dsonar.login=$SONAR_LOGIN --refresh-dependencies --no-daemon"
			}
		}
	}
},
springio: {
	stage('Spring IO') {
		node {
			checkout scm
			try {
				sh "./gradlew clean springIoCheck  --refresh-dependencies --no-daemon --stacktrace"
			} finally {
				junit '**/build/spring-io*-results/*.xml'
			}
		}
	}
}

stage('OSSRH Deploy') {
	node {
		if(currentBuild.result == 'SUCCESS') {
			checkout scm
			withCredentials([file(credentialsId: 'spring-signing-secring.gpg', variable: 'SIGNING_KEYRING_FILE')]) {
				withCredentials([string(credentialsId: 'spring-gpg-passphrase', variable: 'SIGNING_PASSWORD')]) {
					withCredentials([usernamePassword(credentialsId: 'oss-token', passwordVariable: 'OSSRH_PASSWORD', usernameVariable: 'OSSRH_USERNAME')]) {
						sh "./gradlew uploadArchives -Psigning.secretKeyRingFile=$SIGNING_KEYRING_FILE -Psigning.keyId=$SPRING_SIGNING_KEYID -Psigning.password=$SIGNING_PASSWORD -PossrhUsername=$OSSRH_USERNAME -PossrhPassword=$OSSRH_PASSWORD  --refresh-dependencies --no-daemon"
					}
				}
			}
		}
	}
}

stage('Deploy Docs') {
	node {
		if(currentBuild.result == 'SUCCESS') {
			checkout scm
			withCredentials([file(credentialsId: 'docs.spring.io-jenkins_private_ssh_key', variable: 'DEPLOY_SSH_KEY')]) {
				sh "./gradlew deployDocs -PdeployDocsSshKeyPath=$DEPLOY_SSH_KEY -PdeployDocsSshUsername=$SPRING_DOCS_USERNAME --refresh-dependencies --no-daemon --stacktrace"
			}
		}
	}
}

stage('Deploy Schema') {
	node {
		if(currentBuild.result == 'SUCCESS') {
			checkout scm
			withCredentials([file(credentialsId: 'docs.spring.io-jenkins_private_ssh_key', variable: 'DEPLOY_SSH_KEY')]) {
				sh "./gradlew deploySchema -PdeployDocsSshKeyPath=$DEPLOY_SSH_KEY -PdeployDocsSshUsername=$SPRING_DOCS_USERNAME --refresh-dependencies --no-daemon --stacktrace"
			}
		}
	}
}

pipeline {
	agent none
	stages {
		
		stage('OWASP Dependency-Check Vulnerabilities') {
					steps {
						dependencyCheck additionalArguments: ''' 
									-o './'
									-s './'
									-f 'ALL' 
									--prettyPrint''', odcInstallation: 'OWASP Dependency Check'
						
						dependencyCheckPublisher pattern: 'dependency-check-report.xml'
					}
			}


		stage('Integration UI Test') {
			parallel {
				stage('Deploy') {
					agent any
					steps {
						sh './jenkins/scripts/deploy.sh'
						input message: 'Finished using the web site? (Click "Proceed" to continue)'
						sh './jenkins/scripts/kill.sh'
					}
				}
				stage('Headless Browser Test') {
					agent {
						docker {
						       image 'maven' 
						       args '-u root' 
						}
					}
					steps {
						sh 'mvn -B -DskipTests clean package'
						sh 'mvn test'
					}
					post {
						always {
							junit 'target/surefire-reports/*.xml'
						}
					}
				}
			}
		}
	}
}



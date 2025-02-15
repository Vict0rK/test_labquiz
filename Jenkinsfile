pipeline {
    agent none

    stages {
        stage('OWASP Dependency-Check Vulnerabilities Lab 6') {
            agent any
            steps {
                dependencyCheck additionalArguments: '''
                    -o './'
                    -s './'
                    -f 'ALL'
                    --prettyPrint''', 
                odcInstallation: 'OWASP Dependency Check'
                
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
		
		stage('Next Generation Warning Plugin Lab 8') {
			agent any 
			steps{
				git branch: 'main', url: 'https://github.com/Vict0rK/ssd_test_labquiz.git'
				sh '/var/jenkins_home/apache-maven-3.9.8/bin/mvn --batch-mode -V -U -e clean verify -Dsurefire.useFile=false -Dmaven.test.failure.ignore'
				sh '/var/jenkins_home/apache-maven-3.9.8/bin/mvn --batch-mode -V -U -e checkstyle:checkstyle pmd:pmd pmd:cpd findbugs:findbugs'

			}
			post {
				always {
				junit testResults: '**/target/surefire-reports/TEST-*.xml'

				recordIssues(enabledForFailure: true, tools: [
					mavenConsole(),
					java(),
					javaDoc()
				])
				recordIssues(enabledForFailure: true, tool: checkStyle())
				recordIssues(enabledForFailure: true, tool: spotBugs(pattern: '**/target/findbugsXml.xml'))
				recordIssues(enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml'))
				recordIssues(enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml'))
				}
			}
		}

		stage('SonarQube Analysis Lab 9'){
			agent any
			steps{
				git branch: 'main', url: 'https://github.com/Vict0rK/ssd_test_labquiz.git'
				script {
                    // Define the SonarQube scanner tool
                    def scannerHome = tool 'SonarQube'

                    // Run the SonarQube scanner
                    withSonarQubeEnv('SonarQube') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=OWASP \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://192.168.6.53:9000 \
                        -Dsonar.token=sqp_9825b4abf7bb1922a8224832734876643f080194
                        """
                    }
                }
			}
			post {
				always {
					recordIssues enabledForFailure: true, tool: sonarQube()
				}
			}
		}

        stage('Integration UI Test Lab 7b') {
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
        stage('Composer Build and Test') {
            agent {
                docker {
                    image 'composer:latest'
                    args '--entrypoint=""'
                }
            }
            stages {
                stage('Build') {
                    steps {
                        sh 'composer install'
                    }
                }
                // if you want to include this, you need to create the phpunit in a test folder (refer to 7a's repo)
                // stage('Test') {
                //     steps {
                //         sh './vendor/bin/phpunit --log-junit logs/unitreport.xml -c tests/phpunit.xml tests'
                //     }
                // }
            }
        }
    }
}


pipeline {
    agent {label "master"}
    tools {
        maven "maven"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.17.0.4:8081"
        NEXUS_REPOSITORY = "jenkin_repo"
        NEXUS_CREDENTIAL_ID = "nexus_cred"
    }
    stages {
        stage("Checkout") {
            steps {

                    cleanWs()
                    git branch: 'master', url: 'https://github.com/shyamalhazra/maven_java_web_example.git'

            }
        }
        stage("Test-Build") {
            steps {
               sh "mvn compile"
               sh "mvn test"
                junit '**/target/surefire-reports/*.xml'
                junit testResults: '**/target/surefire-reports/*.xml'
            }
            post{
            failure{
                mail bcc: '', body: '''Hi aaa,
                Recent build has failed. Have a look.

                Thanks,''', cc: '', from: 'shyamanalytic@gmail.com', replyTo: '', subject: 'Build Failure', to: 'shyamcloud02@gmail.com'
            }
        }
        }
        stage('Static Code analysis') {
	        steps{
            withSonarQubeEnv('sonarqube') {
            sh 'mvn  package sonar:sonar'
    } // SonarQube taskId is automatically attached to the pipeline context
    script{
    sleep(60)
    timeout(time: 5, unit: 'MINUTES') {
    def qg = waitForQualityGate()
    print "Finished waiting"
    if (qg.status != 'OK') {
        error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
}
}
  }
  post{
    failure{
                
                mail bcc: '', body: '''Hi aaa,
                Recent build Quality gate status failed. Have a look.

                Thanks,''', cc: '', from: 'shyamanalytic@gmail.com', replyTo: '', subject: 'Quality gate failed', to: 'shyamcloud02@gmail.com'
            }
    success{
        script {    sh "mvn package -DskipTests=true"
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                        groupId: pom.groupId;
                        version: pom.version;
                        echo "${pom.groupId}"
                         build wait: false , job: 'demo_ppl_2', parameters: [string(name: 'groupId', value: "${pom.groupId}"), string(name: 'artifactId', value: "${pom.artifactId}"), string(name: 'version', value: "${pom.version}"), string(name: 'packaging', value: "${pom.packaging}")]
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                    
                    //echo "copying pom.xml to downstream job "
					//archiveArtifacts artifacts: 'pom.xml', onlyIfSuccessful: true
                }
    }
    }
  }
  
}
}

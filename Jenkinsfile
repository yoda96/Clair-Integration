node(){
    env.CLAIR_IP="172.28.128.53"
    env.MVN_HOME="/home/vagrant/maven/apache-maven-3.5.4"
    cleanWs()
    stage("CheckOut"){
        checkout scm
    }
    stage("Unit Tests"){
        sh "${env.MVN_HOME}/bin/mvn clean test"
    }
    stage("Vulenrability Analysis"){
        sh "${env.MVN_HOME}/bin/mvn verify"
        emailext    body: '${FILE,path="target/dependency-check-report.html"}',
                    mimeType: 'text/html', subject: 'Library Vulnerability Analysis: ${JOB_NAME} - ${BUILD_NUMBER}', 
                    attachmentsPattern: 'target/dependency-check-report.html',
                    to: 'dnyaneshwar.mundhe@infostretch.com'
    }

    stage("Build Artifact") {
        sh "${env.MVN_HOME}/bin/mvn -DskipTests package"
    }
    
    stage("Push Artifact"){
        sh "echo Pushing the Artifact"
    }

    stage("Build Docker Image"){
        sh "docker build -t infostretch/shipping:latest -f Dockerfile ."
    }

    stage("Docker Image Vulnerability Analysis"){
        sh "clair-scanner -t Critical --ip=${env.CLAIR_IP} -r target/security_report.json infostretch/shipping:latest > /dev/null"
        sh "convertor.py target/security_report.json target/security_report.html"
        emailext    body: '${FILE,path="target/security_report.html"}',
                    mimeType: 'text/html', subject: 'Container Image Vulnerability Analysis: ${JOB_NAME} - ${BUILD_NUMBER}', 
                    attachmentsPattern: 'target/security_report.html',
                    to: 'anshul.patel@infostretch.com'
    }

    stage("Push Docker Image"){
        sh "echo Pushing the Docker Image"
    }
}

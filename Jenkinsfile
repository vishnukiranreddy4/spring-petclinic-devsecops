node {
    def workspace = pwd()
    
    stage('Application_Build') {
        checkout scm
        bat './mvnw clean package -DskipTests'
    }    
    stage('Application_Dependency_Check') {
        bat './mvnw dependency-check:check'        
        dependencyCheckPublisher canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '**/dependency-check-report.xml', unHealthy: ''
    }
    stage('Application_Unit_Test') {        
        bat './mvnw compiler:testCompile surefire:test'
        step([$class: 'JUnitResultArchiver', testResults: "**/surefire-reports/*.xml"])
    }    
    stage('Application_Code_Analysis') {        
        withSonarQubeEnv {
            bat './mvnw sonar:sonar -Dsonar.projectKey=Petclinic_Static_Code_Analysis -Dsonar.projectName=Petclinic_Static_Code_Analysis -PQP1'
        }
    }
    stage('Application_Static_Security_Testing') {        
        withSonarQubeEnv {
            bat './mvnw sonar:sonar -Dsonar.projectKey=Petclinic_SAST -Dsonar.projectName=Petclinic_SAST -PQP2'
        }        
    }
    stage('Application_Deploy') {
        bat "copy ${workspace}\\target\\petclinic.war ${TOMCAT_HOME}\\webapps\\"
    }    
    stage('Application_Dynamic_Security_Testing') {
        sh 'zap-cli --zap-path="${OWASP_ZAP_HOME}" --port=${OWASP_ZAP_PORT} quick-scan -s xss,sqli --spider -r http://localhost:${TOMCAT_PORT}/petclinic'
        sh 'zap-cli --zap-path="${OWASP_ZAP_HOME}" --port=${OWASP_ZAP_PORT} report -o ./ZAP_Report.html -f html'
}
}

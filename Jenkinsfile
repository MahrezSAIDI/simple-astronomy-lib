pipeline {
    agent any
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "http://localhost:8081/"
        NEXUS_REPOSITORY = "maven-nexus-repo"
        NEXUS_CREDENTIAL_ID = "nexus-user-credentials"
    }
    stages {
        stage('test') {
            steps {
                bat 'mvn clean'
                bat 'mvn test' 
            }
        }
        stage('Package') {
            steps {
                bat 'mvn clean'
                bat 'mvn package' 
            }
        }
        stage('Analyse') {
            steps {
                bat 'mvn checkstyle:checkstyle'
                bat 'mvn spotbugs:spotbugs'
                bat 'mvn pmd:pmd' 
            }
        }
    }
    
    post {
        always {
            junit '**/surefire-reports/*.xml'
            
            recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), javaDoc()]
            recordIssues enabledForFailure: true, tool: checkStyle()
            recordIssues enabledForFailure: true, tool: spotBugs()
            recordIssues enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml')
            recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml')
            
        }

    }

}
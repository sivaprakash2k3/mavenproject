pipeline {
    agent any

    tools {
        jdk 'JDK-21'
        maven 'maven_home'
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'test', 'prod'],
            description: 'Select deployment environment'
        )

        string(
            name: 'APP_VERSION',
            defaultValue: '1.0.0',
            description: 'Application version'
        )

        booleanParam(
            name: 'RUN_TESTS',
            defaultValue: true,
            description: 'Run unit tests'
        )
    }

    stages {

        stage('Tool Check') {
            steps {
                bat 'java -version'
                bat 'mvn -version'
                bat 'git --version'
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Workspace Check') {
    steps {
        bat 'echo WORKSPACE=%WORKSPACE%'
        bat 'dir'
    }
}

        stage('Build') {
            steps {
                echo "Building version: ${params.APP_VERSION}"
                echo "Environment: ${params.ENVIRONMENT}"

                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            when {
                expression {
                    params.RUN_TESTS
                }
            }

            steps {
                bat 'mvn test'
            }
        }
        stage('SonarQube Analysis') {
            steps {
        withCredentials([
            string(
                credentialsId: 'sonarqube-token',
                variable: 'SONAR_TOKEN'
            )
        ]) {
            withSonarQubeEnv('SonarQube-EC2') {
                bat 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.token=%SONAR_TOKEN%'
            }
        }
    }
}
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                    fingerprint: true
            }
        }
    }
}

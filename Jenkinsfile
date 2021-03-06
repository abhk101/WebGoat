pipeline { 
    agent any
    tools {
      maven 'M3'
      jdk 'jdk8'
    }
    stages {
        stage ('Build') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    mvn -B -DskipTest=true install
                ''' 
            }
            post {
                always {
                    junit '**/target/surefire-reports/**/*.xml' 
                }
            }
        }
        stage('Scan App - Build Container') {
            steps{
                parallel('IQ-BOM': {
                    nexusPolicyEvaluation failBuildOnNetworkError: false, 
                    iqApplication: 'webgoat8', 
                    iqStage: 'build', 
                    iqScanPatterns: [[scanPattern: '']], 
                    jobCredentialsId: ''
                 },
                 'Static Analysis': {
                    echo '...run SonarQube or other SAST tools here'
                 },
                 'Build Container': {
                    sh '''
                        cd webgoat-server
                        mvn -B docker:build
                    '''
                 })
            }

        }
        stage('Test Container') {
            steps{
                echo '...run container and test it'
            } 
            post {
                success {
                    echo '...the Test Scan Passed!'
                }
                failure {
                    echo '...the Test  FAILED'
                    error("...the Container Test FAILED")
                }
            }   
        }
        stage('Scan Container') {
            steps{
                sh "docker save mycompany.com:18444/webgoat/webgoat-8.0 -o ${env.WORKSPACE}/webgoat.tar"

                nexusPolicyEvaluation failBuildOnNetworkError: false, 
                iqApplication: 'webgoat8', 
                iqStage: 'release', 
                iqScanPatterns: [[scanPattern: '*.tar']], 
                jobCredentialsId: ''
            } 
            post {
                success {
                    echo '...the IQ Scan PASSED :D'
                }
                failure {
                    echo '...the IQ Scan FAILED :('
                    error("...the IQ Scan FAILED")
                }
            }   
        }
        stage('Publish Container') {
            when {
                branch 'master'
            }
            steps {
                sh '''
                    docker tag webgoat/webgoat-8.0 mycompany.com:18444/webgoat/webgoat-8.0:8.0
                    docker push mycompany.com:18444/webgoat/webgoat-8.0
                '''
            }
        }
    }
}
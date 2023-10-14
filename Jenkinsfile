pipeline{
    
    agent any
    
    tools{
        
        jdk 'jdk17'
        maven 'maven3'
        
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages{
        
        stage('clean workspace'){
            
            steps{
                cleanWs()
            }
        }
        
        stage('checkout SCM'){
            
            steps{
                
                git 'https://github.com/amolghanwat09/jpetstore-6.git'
            }
        }
        
        stage('maven complie'){
            
            steps{
                
                sh 'mvn clean compile'
            }
        }
        
        stage('maven test'){
            
            steps{
                
                sh 'mvn test'
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=petstore \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=petstore '''
                }
            }
        }
        stage("quality gate"){
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
           }
        }
        
        stage ('Build war file'){
            steps{
                sh 'mvn clean install -DskipTests=true'
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        
        stage('AnsiblePlaybook Docker') {
            steps {
                dir('Ansible'){
                  script {
                         ansiblePlaybook credentialsId: 'ssh', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'docker.yaml'
                        }     
                   }    
              }
        }
        stage('k8s using ansible'){
            steps{
                dir('Ansible') {
                    script{
                        ansiblePlaybook credentialsId: 'ssh', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'kube.yaml'
                    }
                } 
            }
        }


        
        
    }
}

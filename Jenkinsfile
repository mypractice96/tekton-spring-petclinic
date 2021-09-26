pipeline {
    agent any
    tools 
	{
        maven "maven"
        jdk "jdk11"
        
    }    
    triggers {
        pollSCM '* * * * *'
    }

    stages {
        
        /* ---------- Development Stages ----------- */
        stage('Dev Pull Code') {
            when { branch 'dev' }
            steps {
                git branch: 'dev', url: 'https://github.com/mypractice96/java-spring-petclinic.git'
            }
        }
        stage('Unit Test') {
            when { branch 'dev' }
            steps {
                sh 'mvn test -Dcheckstyle.skip'
            }
        }
        stage('Publish Unit Test Results') {
            when { branch 'dev' }
            steps {
                junit 'target/surefire-reports/*.xml'
                publishCoverage adapters: [jacocoAdapter('**/target/site/jacoco/*.xml')], sourceFileResolver: sourceFiles('NEVER_STORE')
                jacoco changeBuildStatus: false, runAlways: true, skipCopyOfSrcFiles: false, sourceInclusionPattern: '**/*.java', sourcePattern: 'src/main/java'           
            }
        }
        stage('Code Quality') {
           when { branch 'dev' }
           steps{
               withSonarQubeEnv('sonarqube-server') {
                 sh "/opt/sonar-scanner/bin/sonar-scanner"
               }
            }
        }
        stage('Wait For Quality Gate') {
           when { branch 'dev' }
           steps{               
             script{
              timeout(time:1, unit: 'MINUTES'){
                  def qg = waitForQualityGate()
                  if(qg.status != 'OK') {
                      error "Pipeline aborted due to Quality Gate failure with status : ${qg.status}"
                  }
              }
              }
           }
        }
        stage('Dev Build') {
           when { branch 'dev' }
           steps{
               sh 'mvn clean install -Dcheckstyle.skip'
               sh 'docker build -t petclinic-dev:${BUILD_NUMBER} .'
            }
        }
        stage('Deploy to Dev') {
           when { branch 'dev' }
           steps{
               sh 'docker rm -f petclinic-dev || true'
               sh 'docker run -d --name petclinic-dev -p 7001:8080 petclinic-dev:${BUILD_NUMBER}'
            }
        }
        
        
        /* ---------- QA Stages ---------- */
        
        stage('QA Pull Code') {
            when { branch 'qa' }
            steps {
                git branch: 'qa', url: 'https://github.com/mypractice96/java-spring-petclinic.git'
            }
        }
        stage('QA Build') {
            when { branch 'qa' }
            steps {
                sh 'mvn clean install'
                sh 'docker build -t petclinic-qa:${BUILD_NUMBER} .'
            }
        }
        stage('Software Component Analysis') {
             when { branch 'qa' }
              steps{
                dependencyCheck additionalArguments: '--scan=./ --format XML --project \'petclinic\' --proxyserver \'172.31.37.11\' --proxyport \'3128\'', odcInstallation: 'dependency-check'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'                  
              }
         }
        stage('SCA Approval') {
           when { branch 'qa' }
           steps{
               input "SCA results verified?"
            }
        }
        stage('Deploy to QA') {
           when { branch 'qa' }
           steps{
               sh 'docker rm -f petclinic-qa || true'
               sh 'docker run -d --name petclinic-qa -p 7002:8080 petclinic-qa:${BUILD_NUMBER}'
            }
        }
        
        stage('Functional Testing') {
           when { branch 'qa' }
           steps{
               build 'selenium tests'
           }
        }
        
        stage('Load Tests') {
            when { branch 'qa' }
           steps{
               sh '/opt/jmeter/bin/jmeter.sh -Jjmeter.save.saveservice.output_format=xml -n -t petclinic-loadtest.jmx -l jmeter.jtl'
               perfReport 'jmeter.jtl'
               
           }
        }
        stage('DAST') {
           when { branch 'qa' }
           steps{
               sh '/opt/ZAP/zap.sh -cmd -quickurl https://hackthebox.eu -quickprogress -quickout ${WORKSPACE}/zap-report.xhtml'
               archiveArtifacts artifacts: 'zap-report.xhtml', onlyIfSuccessful: true
            } 
        }
        
        stage('Notify User') {
           when { branch 'qa' }
           steps{
               sh 'echo Sending mail...Mail Sent'
            }
        }
        
        
        /* -------- Production Stages -------- */ 
               
        
        stage('Prod Pull Code') {
            when { branch 'main' }
            steps {
                git branch: 'main', url: 'https://github.com/mypractice96/java-spring-petclinic.git'
            }
        }
        stage('Prod Build') {
            when { branch 'main' }
            steps {
                sh 'mvn clean install'
                sh 'docker build -t petclinic:${BUILD_NUMBER} .'
            }
        }
        stage('Deploy to Prod') {
           when { branch 'main' }
           steps{
               sh 'docker rm -f petclinic || true'
               sh 'docker run -d --name petclinic -p 7003:8080 petclinic:${BUILD_NUMBER}'
            }
        }

               
         
     }
}  

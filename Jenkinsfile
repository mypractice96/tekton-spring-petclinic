pipeline {
    agent any
    stages {
        
        stage('dev-deploy') {            
            when {
                branch 'dev'  
            }            
            steps {

                sh 'echo Deployed to Dev'                
                sh 'cat abc.txt'
            }
       }
        
        stage('test-deploy') {            
            when {
                branch 'test'  
            }            
            steps {
                sh 'echo Deployed to QA'                
                sh 'cat abc.txt'
            }
       }  
        
         
        stage('prod-deploy') {            
            when {
                branch 'main'  
            }            
            steps {
                sh 'echo Deployed to Prod'               
                sh 'cat abc.txt'
            }
       }

               
         
     }
}  

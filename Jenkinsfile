pipeline {
    agent any

    environment {
        // Ortam değişkenleri (Windows için çift ters eğik çizgi)
        MAVEN_HOME = 'C:\\Program Files\\Apache\\Maven\\apache-maven-3.9.5'
    }

    stages {
        stage('Checkout') {
            steps {
                // GitHub reposundan kodu çeker.
                git 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }
        
        stage('Build') {
            steps {
                // Maven ile projeyi derler (Windows için 'bat' komutu).
                bat "${MAVEN_HOME}\\bin\\mvn clean package"
            }
            post {
                failure {
                    // Build başarısız olduğunda çalışır.
                    script {
                        // Hata logunu dosyaya yazar.
                        def surefireReports = findFiles(glob: 'target/surefire-reports/*.txt')
                        def errorLog = surefireReports ? readFile(surefireReports[0].path) : 'No error log found.'
                        writeFile file: 'errorLog.txt', text: errorLog
                    }
                }
            }
        }
        
        stage('Analyze Failure') {
            when {
                // Sadece build başarısız olduğunda çalışır.
                expression { currentBuild.currentResult == 'FAILURE' }
            }
            steps {
                script {
                    // OpenAI API anahtarını güvenli bir şekilde yükler
                    withCredentials([string(credentialsId: 'openai-api-key', variable: 'OPENAI_API_KEY')]) {
                        // Hata logunu OpenAI API'sine göndermek için Python script'ini çağırır (Windows için 'bat' komutu).
                        bat "python analyze_failure.py errorLog.txt openai_response.txt %OPENAI_API_KEY%"
                    }
                }
            }
        }
    }

    post {
        always {
            // Build durumundan bağımsız olarak her zaman çalışır.
            cleanWs() // Workspace'i temizler.
        }
    }
}

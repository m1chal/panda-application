pipeline {
    agent {
      label 'ubuntucompose'
    }

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
    }
    
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }

    stages {
        
        stage('Clear running apps') {
            steps {
                // sh "docker rm -f pandaapp || true"
                sh """
                docker rm -f ${NAZWA_KONTENERA} || 
                 {
                    echo "This container is not existing" 
                 }
                 """
                // gdzie nazwa kontenera to "pandaapp"
                // sh "if [[ $? = 1 ]] then echo 'BLAD'" - sprawdzic
                
            }
        }
        
        stage('Get Code') {
            steps {
                git branch: 'selenium_grid', url: 'https://github.com/m1chal/panda-application.git'
            }
        }        
        
        stage('Build and Junit') {
            steps {
                sh "mvn clean install"
            }
        }
        
        stage('Build Docker image') {
            steps {
                sh "mvn package -Pdocker"
            }
        }
        
        stage('Run Docker app') {
            steps {
                // echo "run step here"
                sh "docker run -d -p 0.0.0.0:8080:8080 --name ${NAZWA_KONTENERA} ${IMAGE}:${VERSION}"
            }
        }
        
        stage('Test Selenium') {
            steps {
                sh "mvn -Pselenium test"
            }
        }
        
        stage('Deploy jar to artifactory') {
            steps {
                configFileProvider([configFile(fileId: '4660bdf8-3eb4-4bd9-a53a-277b8902c143', targetLocation: 'https://github.com/m1chal/panda-application.git', variable: 'mavensettings')]) 
                {
                    sh "mvn -s $mavensettings deploy"
                }
            }
        }

    }
    
    post {
        always {
            sh "docker stop ${NAZWA_KONTENERA}"
            deleteDir()
        }
            
    }
}

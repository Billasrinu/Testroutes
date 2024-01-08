def img
pipeline {
    environment {
        registry = " Billa/python-jenkins" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'DOCKERHUB'
        githubCredential = 'GITHUB'
        dockerImage = ''
    }
    agent any
    stages {
        
        stage('checkout') {
                steps {
                git branch: 'main',
                credentialsId: githubCredential,
                url: 'https://github.com/Billasrinu/Testroutes.git'
                }
        }

        stage('Setup'){
       steps{
        dir('.'){
            sh 'python3.8 -m venv ./venv'
        }            
        }
     }    


       stage('Unit Tests'){ 
           steps{
             dir('.') {
                 sh '. ./venv/bin/activate'
                 sh 'pip install -r requirements.txt'
                 sh 'pytest -v --junitxml=docs/unit-tests/htmlcoverage/coverage.xml --cov-report xml --cov app.main'
             }           
            }                    
         }                


        
       stage('Publish Test Report'){ 
           steps{
              cobertura autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: 'coverage*.xml', conditionalCoverageTargets: '70, 0, 0', failUnhealthy: false, failUnstable: false, lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0', onlyStable: false, sourceEncoding: 'ASCII', zoomCoverageChart: false
              archiveArtifacts artifacts: 'docs/unit-tests/htmlcoverage/*.*'
            }                    
         } 



        
        stage ('Test'){
                steps {
                sh "pytest testRoutes.py"
                }
        }
        
        stage ('Clean Up'){
            steps{
                sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${JOB_NAME} | awk \'{print $1}\')'
                sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force' //this will delete all images
                sh returnStatus: true, script: 'docker rm ${JOB_NAME}'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    img = registry + ":${env.BUILD_ID}"
                    println ("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }

        stage('Push To DockerHub') {
            steps {
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com ', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
                    
        stage('Deploy') {
           steps {
                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 5000:5000 ${img}"
          }
        }

      }
    }

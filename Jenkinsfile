pipeline {
  agent any


  environment {
       
       IMAGE_NAME = "twitter-static-website"
       IMAGE_TAG = "v5"
       STAGING = "abderrezak-prod"
      
     }



  stages{

  stage('Test connexion') {
           agent any
           steps {
              script {
               echo "helloworld"
              }
           }
      }
  




 stage('Build image') {
           agent any
           steps {
              script {
               sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
              }
           }
      }


  stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    docker run --name $IMAGE_NAME -d -p 80:80 -e PORT=80 $IMAGE_NAME:$IMAGE_TAG
                    sleep 5
                 '''
               }
            }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                    curl http://jenkins | grep -i "Linux Tweet App"
                '''
              }
           }
      }
      stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
                 docker stop $IMAGE_NAME
                 docker rm $IMAGE_NAME
               '''
             }
          }
     }



      stage ('Login and Push Image on a local registry') {
          agent any
          steps {
             script {
               sh '''
                   
                   docker tag $IMAGE_NAME:$IMAGE_TAG jenkins:5000/groupe2/$IMAGE_NAME:$IMAGE_TAG
    
                   docker push jenkins:5000/groupe2/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
      }     
     
     
     stage('Prepare ansible environment') {
            agent any
            environment {
                PRIVATE_KEY = credentials('private_keys_jenkins')
            }
            steps {
                sh '''
                     cp  $PRIVATE_KEY  id_rsa
                     sudo chmod 600 id_rsa
                '''
            }
     }    




    stage('Push image in staging and deploy it') { 
           agent any
           steps {
               script {
                 sh '''
                    cd $WORKSPACE/ansible && ansible-playbook playbooks/deploy_app.yml  --private-key ../id_rsa                 
                 '''
               }
           }
     }
    
    
    stage('Push image in prod and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/main' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('heroku_api_key')
      }  
      steps {
          script {
            sh '''
              cd $WORKSPACE/
              heroku container:login
              heroku create $STAGING || echo "project already exist"
              heroku container:push -a $STAGING web
              heroku container:release -a $STAGING web
            '''
          }
        }
     }
  
    
    
    
    


  }


  post {
    always {
      discordSend description: 'Jenkins Pipeline Build', webhookURL: 'https://discord.com/api/webhooks/971457836276121670/Z-aLx7cJULq_p9nwlXhqOldM_fiecpxgmuvNdubxHxqha-uvkFhlXO3XW_mytk_GmYmj'
    }
  }

}

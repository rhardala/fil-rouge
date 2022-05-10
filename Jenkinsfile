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

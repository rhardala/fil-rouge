# Presentation du projet 

Le but de travail est de réaliser une pipeline pour le déploiement d'un site web statique dans un environnement de staging et un environnement de prod. Il s'agit de :

- Déployer une VM Centos appelée **jenkins_baremetal**  dans laquelle on installe **Jnekins** , **docker** et **Ansible** à l'aide d'un **Vagrantfile**. 
- Déployer une VM appelée Centos **worker1** pour environnement de **stating** à l'aide du même **Vagrantfile**.
- Une machine sur le cloud **Heroku** pour environnement de **prod** 

# Architecture

Dans le présent projet , nous allons déployer l'architecture suivante:


![Architecture](/Images/architecture.png "Architecture")


# Mise en place de l'architecture

## Lancement des Vm et création d'un registry local

- On lance de déploiement des VM à l'aide de Vagrantfile.
     ```
    vagrant up --provision
    ``` 
- Une fois les deux Vmx sont installées , on se connecte sur le serveeur jenkins.
  
    ```
    vagrant ssh jenkins_baremetal
    ```
- A l'aide du docker-compose.yml , on crée un registry local et son IHM sous forme de contenaires docker. Ces deux derniers écoutent respectivement sur les port 5000 et 8080. Le registry local est un répo dans lequel on sauvegarde les images docker qu'on crée. Il est équivalent à une version local de dockerub.

     ```
    vi docker-compose.yml
    ```
- On copie le contenu du fichier fourni dans ce fichier.
- On crée un volume persistent pour pouvoir stocker nos images docker. ce volume est appelé volume. On monte le volume du contenaire dans ce dernier.

    ```
    sudo mkdir ./volume
    ```
- Pour pouvoir accéder au registre via https , on a besoin de crée un fichier de configuration.
  
```
    sudo vi /etc/docker/daemon.json
```
- On colle la ligne suivante 

```
    
{
     "insecure-registries":["jenkins:5000"]
}
```
- On redémarre le service docker

```
    sudo service docker stop
    sudo service docker start
```


- On lance les deux contenaire.
  
    ```
    docker-compose -f docker-compose.yml up -d
    ```
-  On consomme notre registry via son IHM. Sur le navigateur on tape l'url 

```
    192.168.99.10:8081
```

![local_registry](/Images/local_registry.PNG "local_registry")



A ce niveau , l'installation du registry est terminée.
Dans cette partie nous allons créer un registry local et une IHM pour pouvoir y accéder:

## Installation des plugin jenkins 

- Depuis le navigateur , on accéde au serveur Jenkins via l'url 

```
    192.168.99.10:8080
```
- On génère le mot de passe dans la machine jenkins

```
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
- On installe les plugin suggérés.
- On crée le premier utilisateur avec les identifiants admin/admin
- Dans Administration Jenkins >Gestion des Plugins >Disponible :  On installe les plugins :Blue ocean , Discord notifier, ansible et on remémarre.

# Mise en place de la Pipeline CI

Le fichier Jenkinsfile décrit les différentes étapes de la pipiline, elles sont dans l'odre:

- Récuperer le code depuis Github.
- Builder une Image depuis le dockerfile fourni.
- Lancer un conteneaire
- Tester l'image.
- Pusher l'image dans le local registry.
- Supprimer les contenaires. 
- Envoyer une notification sur discord à la fin de la CI.

Ces étapes sont representé en vert dans l'image illutrant l'architecture.

Pour crée la pipeline , on va dans nouveau Item >Pipeline , on nomme notre pipeline test_CI on clique sur Ok.

![name_pipeline](/Images/name_pipeline.PNG "name_pipeline")

- Dans général on cohe Git Project et on ajoute l'url de notre projet.

```
    https://github.com/abderrezakaddar/fil-rouge
```
- Dans Pipeline on choisit Pipeline From SCM
- Dans SCM on choisit Git
- Dans notre projet sur Githb , on clique sur code et on copie l'url HTTP et on la colle dans jenkins dans REgistry URL.
- On spécifie la branche avec laquelle on travail , dans notre cas c'est main.
- Le Script path on le laisse Jenkinsfile par defaut.
- On sauvegarde et on lance le build de la pipeline.

![pipelineCI](/Images/pipelineCI.PNG "pipelineCI")

- On visualise les étapes de notre pipeline dans Blue Ocean.

![pipelineCI01](/Images/pipelineCI01.PNG "pipelineCI01")

- Sur l'IHM de notre registry , on voit que l'image a été bien pushé.

![plocal_registryCI](/Images/local_registryCI.PNG "local_registryCI")

- Une notification a été bien envoyée sur Discord. 

# Mise en place de la Pipeline CD


### Configuration d'Heroku


### Création des credentials pour heroku

- Dans Administration de Jenkins > Manage Credentials > Global > Ajouter des identifiants > on ajoute un secret Text. 
- Dans Heroku > Account Setings > on déroule en bas API Key  Revel : on génère le **secret text** et on l'ajoute dans le credential Jenkins. 

![heroku_credential](/Images/heroku_credential.PNG "heroku_credential")



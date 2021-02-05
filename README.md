# Devops_rendu
rendu tp, cours devops



# TP 1

Pour creer notre propre image docker nous aurons besoin d'un dockerfile
### creation de mon dockerfile: exemple d'une image postgres 
* creer un fichier nommé 'Dockerfile' (sans extension) 
* dans le fichier dockerfile je met:
````
FROM postgres:11.6-alpine
ENV POSTGRES_DB=db \
	POSTGRES_USER=sagar \
	POSTGRES_PASSWORD=sagar
COPY *.sql /docker-entrypoint-initdb.d/
````
liste d'instructions possible dans le Dockerfile:
- FROM: permettre de definir limage utilisée comme base de notre image(image source)
- ADD : copier du contenu depuis la machine vers limage docker en crs de construction
- CMD : la commande qui sera lancer par le conteneur lors de son execution 
- VOLUME : permet d'indiquer quel répertoire vous voulez partager avec votre host.
- WORKDIR : specifier le répertoir courant 
- COPY : copier du contenu depuis la machine vers le conteneur [src,... dest]
- ENV : definir les variables d'environnement qui sera utilisé
- EXPOSE: permet de définir les ports d'écoute par défaut (le port qui sera exposer coté conteneur)
- USER : specifier l'utilisateur
- RUN qui vous permet d’exécuter des commandes dans votre conteneur ;
etc...

### builder et runner une image: tjrs exemple d'une image postgres
* je cree mes script sql ( creation db et insert data) dans le meme repertoire que mon dockerfile
* ensuite dans un bash je build mon image postgres: ```$ docker build -t sagargueye/postgres .```
* ensuite je run :``` $ docker run --rm --name postgres -d -p 5432:5432 sagargueye/postgres```
    - -d permet de detacher le conteneur du processus actuel et donc de lancer plusieurs compteneur en parallele
    - --rm permet de remove l'image une fois que le conteneur soit arreté
    - -p permet douvrir un port réseau sur la machine vers notre conteneur docker
    - --name donner un nom a notre conteneur
    - sagargueye/posgres cest notre image
* volume:  sauvegarder tes donnees de la base lors de la destruction du conteneur, dans le repos indiquer
    * il faut ajouter dans la commande run l'option -v et le chemin pour lier la base à un volume:
    * ```$ docker run --rm --name postgres -d -p 5432:5432 -v "C:/Users/Utilisateur/Desktop/CPE/semestre 8/Devops/tp1_docker/database:/var/lib/postgresql/data" sagargueye/postgres```

PROBLEME: ya un soucis car postgres et adminer communique pas. 

SOLUTION:
* pour faire communiquer adminer et posgres, je dois les mettre dans le mm reseau 
* creation reseaux: ```$ docker network create mon_reseau```
* lister les reseaux: ```$ docker network ls```
* mettre adminer dans le reseau créé: ```$ docker run --name adminer --network mon_reseau -p 8080:8080  --rm adminer```
* mettre notre bd postgres dans le reseau: ```$ docker run --rm --name postgres --network mon_reseau  -d -p 5432:5432 sagargueye/postgres```

### Testing: on va se connecter sur adminer pour verifier si tout est ok (presence de notre base de données et de nos données)
pour se connecter au adminer postgres: http://localhost:8080/adminer/
* systeme: postgresSql
* serveur: postgres
* bd_name: db
* user: sagar
* pwd: sagar

### exemple 2: application JAVA qui affiche "hello world"
* Dockerfile:
    ```
    FROM openjdk:7
    COPY C:\Users\Utilisateur\Desktop\tp1_docker
    WORKDIR C:\Users\Utilisateur\Desktop\tp1_docker
    RUN javac Main.java
    CMD ["java", "Main"]
    ```
* build: ```$ docker build -t sagargueye/openjdk:7 .```
* Run: ```$ docker run --rm --name openjdk sagargueye/openjdk:7```

### exemple 3 : java multi stage
* dockerfile:
    ```
    # Build
    FROM maven:3.6.3-jdk-11 AS myapp-build
    ENV MYAPP_HOME /opt/myapp
    WORKDIR $MYAPP_HOME
    COPY pom.xml .
    RUN mvn dependency:go-offline
    
    COPY src ./src
    RUN mvn package -DskipTests
    
    # Run
    FROM openjdk:11-jre
    ENV MYAPP_HOME /opt/myapp
    WORKDIR $MYAPP_HOME
    COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
    ENTRYPOINT java -jar myapp.jar
    ```
* creation reseaux: ```$ docker network create monreseau```
* mettre adminer dans le reseau créé: ```$ docker run --name adminer --network mon_reseau -p 8080:8080  --rm adminer```
* mettre notre bd postgres dans le reseau: ```$ docker run --name postgres --network mon_reseau --rm sagargueye/postgres```
* builder notre image: ```$ docker build -t sagargueye/backapi . ```   
* run: ```$ docker run --rm --name backapi -p 8081:8080 --network monreseau sagargueye/backapi```

### http server
* on cree notre index.html dans un repos qui va contenir notre dockerfile
* dockerfile
    ```
    FROM httpd
    COPY ./ /usr/local/apache2/htdocs/
    ```
* ```$ docker run --rm --name mapage -p 8080:80 sagargueye/mapageweb```
* ```$ docker build -t sagargueye/mapageweb .```
* on teste sur : http://localhost:8080/

### Quelles que commandes docker
* docker ps => visualiser lensemble des conteneur en cours de fonctionnement
* docker rm [ID_CONTENEUR] => supprime limage
* docker pull [IMAGE] => recuperer une image depuis dockerhub
* docker stop [ID_CONTENEUR] =>arréter unn conteneur en cours
* docker system prune => delete lensemble des images, conteneur arrétés , volumes et network
* pour tout supprimer d'un seul coup( imageges , conteneur, et volumes): docker system prune -fa --volumes
* run
    * -p 8080 => cest mon localhost sur mon pc
    * :8080 cest le port sur conteneur quon expose 


la fin du tp1 est le debut TP2 ==> docker compose
___
# TP 2

### Docker-compose
docker-Compose est un outil permettant de définir et d'exécuter des applications Docker multi-conteneurs.  
Avec Compose, on utilise un fichier YAML pour configurer les services de notre application. 
Ensuite, avec une seule commande, on crée et démarre tous les services à partir de notre configuration.

* lancer docker-compose: ```$ docker-compose up -d```
* on go to the home page de notre application : http://localhost/
* ensuite dans http://localhost/departments/IRC,  ajouter un student
* restart la base: ```$ docker-compose restart database```
* retourner dans http://localhost/departments/IRC et verifier si les donnees sont tjrs la 
* va checker les conf de appach: ``` nano ./sample-application-students/sample-application-frontend/httpd.conf```


### Travis

Travis CI est un service d'intégration continue utilisé pour construire et tester des projets hébergés sur GitHub et Bitbucket.
Travis CI est configuré en ajoutant un fichier nommé .travis.yml au répertoire racine du dépôt. 
Ce fichier précise le langage de programmation utilisé, l'environnement de construction 
et de test souhaité (y compris les dépendances qui doivent être installées avant que le logiciel 
puisse être construit et testé), et divers autres paramètres.

* utilisation de travis:
    * forker le projet git https://github.com/takima-training/sample-application-students
    * cloner le projet en local
    * creer un fichier .travis.yml
    ```
    git:
      depth: 5
    stages:
      - "Build and Test"
      - "Package"
    jobs:
      include:
        - stage: "Build and Test"
          language: java
          jdk: oraclejdk11
          before_script:
            - "cd sample-application-backend"
          script:
            - "mvn clean verify"
        - stage: "Build and Test"
          language: node.js
          node_js: "12.20"
          before_script:
            - "cd sample-application-frontend"
          script:
            - "npm install"
        - stage: "Package"
          before_script:
            - "cd sample-application-backend"
          script:
            - "docker build -t $ID_DOCKERHUB/sample-application-backend ."
            - "docker login -u $ID_DOCKERHUB -p $PWD_DOCKERHUB"
            - "docker push $ID_DOCKERHUB/sample-application-backend"
        - stage: "Package"
          before_script:
            - "cd sample-application-frontend"
          script:
            - "docker build -t $ID_DOCKERHUB/sample-application-frontend ."
            - "docker login -u $ID_DOCKERHUB -p $PWD_DOCKERHUB"
            - "docker push $ID_DOCKERHUB/sample-application-frontend"
    cache:
      directories:
        - "$HOME/.m2/repository"
        - "$HOME/.npm"
    services:
      - docker
    ```
    * creer une nouvelle branch et commit les changements dessus
    * avant de push, on va creer un compte travis (Auth par sso via github)
    * synchroniser le repos 
    * creer des variables d'environnements avec mes identifiants DOCKERHUB (id et pwd) (dans setting de mon repos travis)
    * via gitbash on push la nouvelle branch et le travis.yml 
    * à ce niveau, on remarquera que travis detecter la nouvelle branche et execute le fichier travis.yml
    * si tout est ok, on remarquera que 2 ripos ont été créé sur notre dockerhub: sample-application-backend et sample-application-frontend
     
___
# TP 3
### Ansible
Ansible est un outil d'automatisation informatique. Il configure des systèmes, déploye des logiciels 
et orchestre des tâches plus avancées telles que des déploiements continus ou des mises à jour sans temps d'arrêt.

Un playbook est un fichier au format YAML qui va donner une liste d'instructions. 
Ces instructions sont passées à Ansible dans l'ordre de leur déclaration.

Role: dans Ansible, le rôle est le principal mécanisme permettant de décomposer un playbook en plusieurs fichiers.
Cela simplifie la rédaction de playbooks complexes et facilite leur réutilisation



### td Ansible
* mise à jour: ```$ sudo apt update```
* ```$ sudo apt install software-properties-common```
* installation de ansible: ```$ sudo apt install ansible```
* ```$ ansible --version```
* ```$ sudo mkdir tp3```
* ```$ cd tp3/```
* on recupere dans notre repertoire tp3, la key private ```$ sudo cp mnt/c/Users/Utilisateur/Desktop/CPE/"semestre 8"/Devops/tp_ansible/sagar.gueye /tp3/sagar.gueye```
* on lui donne les droits ```$ sudo chmod 400 sagar.gueye``` ou ```$ sudo chmod 700 sagar.gueye``` 
* on utilise la key pour ssh notre server ```$ sudo ssh -i sagar.gueye centos@sagar.gueye.takima.cloud```
* on ajoute notre domaine 'sagar.gueye.takima.cloud' dans le fichier host de ansible: ```$ sudo nano /etc/ansible/hosts```  
* on essaie de painger pour tester la connexion :```$ sudo ansible all -m ping --private-key=sagar.gueye -u centos```
* mise en service de appach (on oublie surtout pas le become pour etre root): ```$ sudo ansible all -m yum -a "name=httpd state=present" --private-key=sagar.gueye -u centos --become```
* creation de notre page index.html: ```$ sudo ansible all -m shell -a 'echo "<html><h1>tient bon sagar cest bientot finis</h1></html>">> /var/www/html/index.html' --private-key=sagar.gueye -u centos --become```
* lancer les services:```$ sudo ansible all -m service -a "name=httpd state=started" --private-key=sagar.gueye -u centos --become```
* check browser ```http://sagar.gueye.takima.cloud/index.html```


### tp Ansible

* forker et cloner le projet git sample-application-students
* ansible/inventories/setup.yml:
    ``` 
    all:
      vars:
        ansible_user: centos
        ansible_ssh_private_key_file: "sagar.gueye"
      children:
        prod:
          hosts: sagar.gueye.takima.cloud
    ``` 
* tester notre inventory par la commande ping: ```$ ansible all -i inventories/setup.yml -m ping ```
* go installer appach: ``` $ ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become```
* Facts: sont des variables non fixés par l'utilisateurs et obtenue par request au serveur. ils sont prefixes par ansible_ . 
exemple demandons la distribution de l'OS de notre serveur : ```$ ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*" ```
* pour runner notre playbook: ``` $ ansible-playbook -i inventories/setup.yml playbook.yml```
* ansible/playbook:
    ``` 
    - hosts: all
      gather_facts: false
      become: yes
    
      tasks:
        - name: Test connexion
          ping:
    ```     
* ansible/deploy-app.yml:
    ```
    - hosts: all
      gather_facts: false
      become: yes
    
      roles:
        - docker
        - create-network
        - create-volume
        - launch-db
        - launch-app
        - launch-front
    ```
* ansible/docker-playbook.yml:
    ```
    - hosts: all
      gather_facts: false
      become: yes
      
      roles:
        - docker
    ```
* connexion ssh ``` $ sudo ssh -i sagar.gueye centos@sagar.gueye.takima.cloud```
* run ```$ sudo ansible-playbook -i ansible/inventories/setup.yml ansible/inventories/playbook.yml```

### annexe tp3: codes
* ansible/roles/docker/task/main.yml:
    ```
    # Install Docker
    - name: Add Docker stable repository
      yum_repository:
        name: docker-ce
        description: Docker CE Stable - $basearch
        baseurl: https://download.docker.com/linux/centos/7/$basearch/stable
        state: present
        enabled: yes
        gpgcheck: yes
        gpgkey: https://download.docker.com/linux/centos/gpg
    
    - name: Install Docker
      yum:
        name: docker-ce
        state: present
    
    - name: Create docker group
      group:
        name: docker
    
    - name: Add centos user do docker group
      user:
        name: centos
        append: yes
        groups:
          - docker
    
    - name: Make sure Docker is running
      service: name=docker state=started
      tags: docker
    
    - name: Enable EPEL repository to be able to install pip
      yum:
        name: epel-release
        state: present
    
    - name: Install pip for Docker Python SDK
      yum:
        name: python-pip
        state: present
    
    - name: Install Docker Python SDK
      pip:
        name: docker
        state: present
    ```
* ansible/roles/create-network/task/main.yml:
    ```
    - name: Create myapp-net network
      docker_network:
        name: myapp-net
        state: present
    ```
* ansible/roles/create-volume/task/main.yml:
    ```
    - name: Create db-volume volume
      docker_volume:
        name: db-volume
        state: present
    ```
* ansible/roles/launch-db/task/main.yml:
    ```
    - name: Launch database container
      docker_container:
        name: database
        state: started
        image: postgres:12.0-alpine
        env:
          POSTGRES_USER: takima
          POSTGRES_PASSWORD: takimapass
          POSTGRES_DB: SchoolOrganisation
        purge_networks: yes
        networks:
          - name: myapp-net
        volumes:
          - db-volume:/var/lib/postgresql/data
    ```
* ansible/roles/launch-app/task/main.yml:
    ```
    - name: Launch backend
      docker_container:
        name: backend
        state: started
        image: ydrevet/simpleapp-back:latest
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://database:5432/SchoolOrganisation
        purge_networks: yes
        networks:
          - name: myapp-net
    ```
* ansible/roles/launch-front/task/main.yml:
    ```
    - name: Launch frontend
      docker_container:
        name: frontend
        state: started
        image: ydrevet/simpleapp-front:latest
        ports:
          - "80:80"
        purge_networks: yes
        networks:
          - name: myapp-net
    ```
* travis.yml:
    ```
    git:
      depth: 5
    
    jobs:
      include:
        - stage: "Build and Test Java"
          language: "java"
          jdk: "oraclejdk11"
          before_script:
            - "cd sample-application-backend"
          script:
            - "mvn clean verify" 
            - "mvn org.jacoco:jacoco-maven-plugin:prepare-agent" 
            - "mvn sonar:sonar -Dsonar.projectKey=$SONARCLOUD_PROJECTKEY"
        - stage: "Build and Test Nodejs"
          language: "node_js"
          node_js: "12.20"
          before_install:
            - "cd sample-application-frontend"
          install:
            - "npm install"
          script:
            - "npm run test"
        - stage: "Publish"
          script:
            - "cd sample-application-backend"
            - "docker build -t $DOCKERHUB_LOGIN/simpleapp-back:latest ."
            - "docker login -u $DOCKERHUB_LOGIN -p $DOCKERHUB_PASSWORD"
            - "docker push $DOCKERHUB_LOGIN/simpleapp-back:latest"
        - stage: "Publish"
          script:
            - "cd sample-application-frontend"
            - "docker build -t $DOCKERHUB_LOGIN/simpleapp-front:latest ."
            - "docker login -u $DOCKERHUB_LOGIN -p $DOCKERHUB_PASSWORD"
            - "docker push $DOCKERHUB_LOGIN/simpleapp-front:latest"
        - stage: "Deploy"
          before_script:
            - "openssl aes-256-cbc -K $encrypted_c02febb1adb2_key -iv $encrypted_c02febb1adb2_iv -in sagar.gueye.enc -out sagar.gueye -d"
            - "chmod 0400 sagar.gueye"
          script:
            - "ssh -i sagar.gueye -o StrictHostKeyChecking=no centos@sagar.gueye.takima.cloud docker stop backend"
            - "ssh -i sagar.gueye -o StrictHostKeyChecking=no centos@sagar.gueye.takima.cloud docker remove backend"
            - "ssh -i sagar.gueye -o StrictHostKeyChecking=no centos@sagar.gueye.takima.cloud docker run --net myapp-net --name backend -d -p 80:80 sagargueye/simpleapp-back"
          on:
            branch: production
    cache:
      directories:
        - "$HOME/.m2/repository"
        - "$HOME/.npm"
    
    addons:
        sonarcloud:
            organization: "sagargueye"
            token: "$SONARCLOUD_TOKEN"
    ```




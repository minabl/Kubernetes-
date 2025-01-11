# Déploiement d’Applications avec Kubernetes

## Objectifs
    Ce projet a pour objectifs :
    
        Découvrir Kubernetes pour l'orchestration de conteneurs.
        
        Déployer une application conteneurisée en utilisant les objets Deployment et Service.
        
        Comprendre le fonctionnement du scaling horizontal et de la mise à jour continue.

## Table des Matières
- [Technologies Utilisées](#technologies-utilisées)
- [Variables d'Environnement](#variables-denvironnement)
- [Configuration de Docker](#configuration-de-docker)
- [Images Docker](#images-docker)
- [Docker Compose](#docker-compose)
- [Comment Exécuter le Projet](#comment-executer-le-projet)

## Technologies Utilisées
Prérequis

Infrastructure :

Windows : Minikube installé et configuré.

Outils installés : kubectl, docker.

Avoir les images Docker suivantes disponibles :

minabf/mern-server:latest

minabf/mern-client:latest

mongo:latest

## Variables d'Environnement
Les variables d'environnement suivantes sont utilisées dans l'application :

- **REACT_APP_API_URL** : Cette variable contient l'URL de base pour le serveur API. Elle est utilisée dans le client React pour faire des requêtes au serveur.
- **MONGO_URI** : L'URI de connexion à MongoDB utilisée par le serveur pour se connecter à l'instance MongoDB.

## Configuration de Docker
Ce projet comprend des Dockerfiles pour le client et le serveur, qui facilitent la construction et l'exécution des services dans des conteneurs isolés. Les configurations incluent :

- **Client** : Un environnement Node.js pour construire l'application React. Les dépendances sont installées et l'application est construite pour une utilisation en production. Un serveur HTTP simple peut être utilisé pour servir l'application construite.
  
- **Serveur** : Un environnement Node.js qui installe les dépendances nécessaires et configure l'application pour écouter sur un port spécifique.
## Étapes Réalisées
  1. Configuration de l'Environnement
     - Vérifiez que votre cluster Kubernetes est opérationnel :
     ````
       kubectl cluster-info
       kubectl get nodes
     ````
      - Lancez Minikube
        ````
          minikube start
        ````
   2. Création des Ressources Kubernetes
      - Déploiement et Service pour MongoDB
          - Fichier mongo-deployment.yaml :
            ````
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: mongo-deployment
            spec:
              replicas: 1
              selector:
                matchLabels:
                  app: mongo
              template:
                metadata:
                  labels:
                    app: mongo
                spec:
                  containers:
                  - name: mongo
                    image: mongo:latest
                    ports:
                    - containerPort: 27017
                    volumeMounts:
                    - name: mongo-data
                      mountPath: /data/db
                  volumes:
                  - name: mongo-data
                    emptyDir: {}

            ````
          - Fichier mongo-service.yaml :
            ````
            apiVersion: v1
            kind: Service
            metadata:
              name: mongo-service
            spec:
              selector:
                app: mongo
              ports:
              - protocol: TCP
                port: 27017
                targetPort: 27017
              type: ClusterIP
            
            ````
       - Déploiement et Service pour l'Application Serveur (Backend)
         
          - Fichier mern-server-deployment.yaml :
            ````
              apiVersion: apps/v1
              kind: Deployment
              metadata:
                name: mern-server-deployment
              spec:
                replicas: 3
                selector:
                  matchLabels:
                    app: mern-server
                template:
                  metadata:
                    labels:
                      app: mern-server
                  spec:
                    containers:
                    - name: mern-server
                      image: minabf/mern-server:latest
                      ports:
                      - containerPort: 5000
                      env:
                        - name: MONGO_URI
                          value: "mongodb://mongo-service:27017/mern-app"

            ````
          - Fichier mern-server-service.yaml :
            ````
            apiVersion: v1
            kind: Service
            metadata:
              name: mern-server-service
            spec:
              selector:
                app: mern-server
              ports:
                - protocol: TCP
                  port: 5000
                  targetPort: 5000 
              type: LoadBalancer
            ````
        - Déploiement et Service pour l'Application client (Frontend)
         
          - Fichier mern-client-deployment.yaml :
            ````
                apiVersion: apps/v1
                kind: Deployment
                metadata:
                  name: mern-client-deployment
                spec:
                  replicas: 3
                  selector:
                    matchLabels:
                      app: mern-client
                  template:
                    metadata:
                      labels:
                        app: mern-client
                    spec:
                      containers:
                      - name: mern-client
                        image: minabf/mern-client:latest
                        ports:
                        - containerPort: 3000
                        env:
                        - name: REACT_APP_API_URL
                          value: "http://127.0.0.1:5000"
            ````
          - Fichier mern-client-service.yaml :
            ````
            apiVersion: v1
            kind: Service
            metadata:
              name: mern-client-service
            spec:
              selector:
                app: mern-client
              ports:
                - protocol: TCP
                  port: 3000
                  targetPort: 3000
              type: LoadBalancer

            ````
      


      
   4. Déploiement des Ressources
      ````
      - kubectl apply -f server-deployment.yaml
      - kubectl apply -f server-service.yaml
      
      - kubectl apply -f client-deployment.yaml
      - kubectl apply -f client-service.yaml

      - kubectl apply -f mongo-deployment.yaml
      - kubectl apply -f mongodb-service.yaml
      
      - kubectl get deployments
      - kubectl get services
      
      ````
      <div>
   <img src="https://github.com/user-attachments/assets/6accfcfd-df74-481e-9db1-ca26bd6d854b" alt="Capture d'écran de l'application React"  height="300" />
 
  </div> 
  Pods 
   <div>
   <img src="https://github.com/user-attachments/assets/7e6515b1-966e-4570-95ca-4626e20e8a05" alt="Capture d'écran de l'application React"  height="300" />
 
  </div> 
      
   4. Accès aux Services
      ````
        minikube service mern-client-service --url
        minikube service mern-server-service --url
      ````
      <div>
   <img src="https://github.com/user-attachments/assets/906de4d9-9f1b-439e-be75-5255b98772f8" alt="Capture d'écran de l'application React"  height="300" />
 
  </div> 
   5. Scaling et Mise à Jour
        ````
        
            kubectl scale deployment mern-server-deployment --replicas=5   
          
      ````
  6. ConfigMap
      ````
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: mern-config
        data:
          REACT_APP_API_URL: "http://127.0.0.1:5000"
          MONGO_URI: "mongodb://mongo-service:27017/mern-app"

      ````
     et modfier le service et le client
     ````
      env:
        - name: REACT_APP_API_URL
          valueFrom:
            configMapKeyRef:
              name: mern-config
              key: REACT_APP_API_URL
     ---
     env:
          - name: MONGO_URI
            valueFrom:
              configMapKeyRef:
                name: mern-config
                key: MONGO_URI
              
     ````
      
  8. Nettoyage des Ressources
      ````
      kubectl delete -f mongo-deployment.yaml
      kubectl delete -f mongo-service.yaml
      kubectl delete -f mern-server-deployment.yaml
      kubectl delete -f mern-server-service.yaml
      kubectl delete -f mern-client-deployment.yaml
      kubectl delete -f mern-client-service.yaml
      ````
  ### Images du Projet

  Voici quelques captures d'écran de l'application :
  <div>
   <img src="https://github.com/user-attachments/assets/da540f63-123a-48ee-a491-bacf10816f5c" alt="Capture d'écran de l'application React" width="300" height="300" />
   <img src="https://github.com/user-attachments/assets/e5e38ce7-b9c0-48e0-9a4a-60a010004fb5" alt="Capture d'écran de l'application React" width="300" height="300" />
   <img src="https://github.com/user-attachments/assets/f4c8ff4e-0c2c-413b-9901-bad315faaa81" alt="Capture d'écran de l'application React" width="300"  height="300"/>
  </div>  
   <div>
   <img src="https://github.com/user-attachments/assets/4ca52970-2d0a-4b47-9ec7-032e05b7bd81" alt="Capture d'écran de l'application React" width="300" />
 
  </div> 
  

## Images Docker
Les images Docker créées pour ce projet sont les suivantes :

- **Image du Client** : `node:lts-alpine`
- **Image du Serveur** : `node:lts-alpine`
- **Image de la Base de Données** : `mongo:latest`

Ces images sont spécifiées dans les Dockerfiles respectifs et sont utilisées lors de la construction et du déploiement des services.

## Docker Compose
Docker Compose est utilisé pour gérer les différents services de l'application, y compris le client, le serveur et MongoDB. Les services sont interconnectés, ce qui permet une communication fluide entre le client et le serveur. Le fichier de configuration spécifie les images, les ports exposés, ainsi que les variables d'environnement nécessaires pour chaque service.

## Comment Exécuter le Projet
1. Assurez-vous d'avoir Docker et Docker Compose installés sur votre machine.
2. Clonez ce dépôt sur votre machine locale.
3. Accédez au répertoire du projet dans votre terminal.
4. Construisez et démarrez l'application en utilisant Docker Compose :

   ```bash
   docker-compose up --build
   ```

5. Accédez au client à [http://localhost:3000](http://localhost:3000).


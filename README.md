# 🚀 Jenkins CI/CD Pipeline — Lab 1

[![Jenkins](https://img.shields.io/badge/Jenkins-2.580.1-red)](https://www.jenkins.io/)
[![Docker](https://img.shields.io/badge/Docker-29.1.3-blue)](https://hub.docker.com/u/arijbelaid)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.31.14-blue)](https://kubernetes.io/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.1.5-green)](https://spring.io/)
[![Maven](https://img.shields.io/badge/Maven-3.9.12-orange)](https://maven.apache.org/)

Pipeline **CI/CD complet** réalisé avec **Jenkins, Maven, Docker, Kubernetes et GitHub**.
Ce projet couvre la mise en place d'un workflow **manuel** puis d'un workflow **automatisé** pour le déploiement d'une application **Spring Boot**.

Le projet a été réalisé sur un **cluster Kubernetes 3 nœuds** (1 master + 2 workers) sous **Ubuntu Server 26.04**.

---

## 📄 Rapport complet

Le rapport détaillé du projet est disponible dans ce dépôt.

---

## 📋 Sommaire

* [1. Présentation](#1-présentation)
* [2. Architecture du cluster](#2-architecture-du-cluster)
* [3. Prérequis](#3-prérequis)
* [4. Partie I — Workflow manuel (sans Jenkins)](#4-partie-i--workflow-manuel-sans-jenkins)
* [5. Création du Dockerfile](#5-création-du-dockerfile)
* [6. Build Maven et Build Docker](#6-build-maven-et-build-docker)
* [7. Push de l'image sur Docker Hub](#7-push-de-limage-sur-docker-hub)
* [8. Déploiement Kubernetes](#8-déploiement-kubernetes)
* [9. Installation de Jenkins](#9-installation-de-jenkins)
* [10. Configuration du job Freestyle `freestyle_demo`](#10-configuration-du-job-freestyle-freestyle_demo)
* [11. Exécution du fichier Java `Test.java`](#11-exécution-du-fichier-java-testjava)
* [12. Récupération du script depuis GitHub](#12-récupération-du-script-depuis-github)
* [13. Déclenchement automatique (H/5)](#13-déclenchement-automatique-h5)
* [14. Préparation de Jenkins pour Kubernetes](#14-préparation-de-jenkins-pour-kubernetes)
* [15. Challenge — Pipeline complet `deploy_spring_k8s`](#15-challenge--pipeline-complet-deploy_spring_k8s)
* [16. Vérification finale](#16-vérification-finale)

---

# 1. Présentation

## 🎯 Objectif

Mettre en place un **pipeline CI/CD complet** pour une application **Spring Boot** en utilisant :

* **Jenkins** : Automatisation du pipeline
* **Maven** : Build de l'application Java
* **Docker** : Conteneurisation de l'application
* **Kubernetes** : Orchestration et déploiement (cluster 3 nœuds)
* **Docker Hub** : Registre d'images Docker
* **GitHub** : Hébergement du code source et des scripts

## 🛠️ Technologies utilisées

| Technologie | Version | Rôle |
|---|---|---|
| Ubuntu Server | 26.04 | Système d'exploitation |
| Java JDK | 17 | Runtime Java |
| Maven | 3.9.12 | Build Java |
| Docker | 29.1.3 | Conteneurisation |
| Kubernetes | 1.31.14 | Orchestration |
| Jenkins | 2.580.1 | Automatisation CI/CD |
| containerd | 2.2.2 | Runtime de conteneurs K8s |
| Flannel | Latest | Réseau CNI Kubernetes |

---

# 2. Architecture du cluster

## 🎯 Objectif

Déployer un **cluster Kubernetes à 3 nœuds** sur des VMs Ubuntu Server.

```text
┌─────────────────────────────────────────────────────────────────┐
│                        CLUSTER KUBERNETES                        │
│                                                                  │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│   │    MASTER    │    │   WORKER 1   │    │   WORKER 2   │     │
│   │192.168.159.15│    │192.168.159.16│    │192.168.159.17│     │
│   │              │    │              │    │              │     │
│   │  - Jenkins   │    │  - Pods      │    │  - Pods      │     │
│   │  - K8s API   │    │  - Flannel   │    │  - Flannel   │     │
│   │  - kubectl   │    │              │    │              │     │
│   └──────────────┘    └──────────────┘    └──────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │
                     ┌────────┴────────┐
                     │   DOCKER HUB    │
                     │ arijbelaid/     │
                     │ jenkins-demo    │
                     └─────────────────┘
📊 Répartition des nœuds
Rôle	Hostname	IP	Fonction
Master	master	192.168.159.15	Control plane + Jenkins
Worker 1	worker1	192.168.159.16	Exécution des pods
Worker 2	worker2	192.168.159.17	Exécution des pods
3. Prérequis
🎯 Objectif
Préparer l'environnement de travail avant de commencer le TP.

VM Ubuntu Server 26.04 (4 Go RAM minimum)

Docker installé

Kubernetes (K3S / Kubeadm) installé

Maven installé

Java JDK 17 installé

Un compte Docker Hub

Un compte GitHub

4. Partie I — Workflow manuel (sans Jenkins)
🎯 Objectif
Découvrir toutes les étapes d'un déploiement manuel :

Cloner le dépôt Spring Boot

Compiler avec Maven

Construire une image Docker

Pousser l'image sur Docker Hub

Déployer sur Kubernetes

🔗 Dépôt source
👉 https://github.com/wahid007/Jenkins-Demo

bash
git clone https://github.com/wahid007/Jenkins-Demo.git
cd Jenkins-Demo
📸 Capture 01 — Clone du dépôt
https://./captures/01-clone-repo.png

Nom de la capture :

text
01-clone-repo.png
5. Création du Dockerfile
🎯 Objectif
Créer un fichier Dockerfile pour conteneuriser l'application Spring Boot.

📄 Contenu du Dockerfile
dockerfile
FROM eclipse-temurin:17-jre
EXPOSE 8080
WORKDIR /app
COPY target/*.jar app.jar
CMD ["java", "-jar", "app.jar"]
📝 Explication
Image de base
dockerfile
FROM eclipse-temurin:17-jre
Utilisation de l'image Eclipse Temurin (JDK 17 léger).

Port exposé
dockerfile
EXPOSE 8080
Le conteneur expose le port HTTP 8080.

Dossier de travail
dockerfile
WORKDIR /app
Tous les fichiers seront copiés dans /app.

Copie du JAR
dockerfile
COPY target/*.jar app.jar
Le JAR généré par Maven est copié dans l'image.

Démarrage de l'application
dockerfile
CMD ["java", "-jar", "app.jar"]
L'application Spring Boot est lancée au démarrage du conteneur.

📸 Capture 02 — Dockerfile
https://./captures/02-dockerfile.png

Nom de la capture :

text
02-dockerfile.png
6. Build Maven et Build Docker
🎯 Objectif
Compiler l'application avec Maven

Construire l'image Docker

🔨 Build Maven
bash
mvn clean install
Résultat attendu
text
[INFO] BUILD SUCCESS
[INFO] Total time:  01:35 min
🐳 Build Docker
bash
docker build -t jenkins-demo:1.0.0 .
Résultat attendu
text
Successfully built 5fec75671049
Successfully tagged jenkins-demo:1.0.0
📸 Capture 03 — Build Maven (BUILD SUCCESS)
https://./captures/03-mvn-build-success.png

Nom de la capture :

text
03-mvn-build-success.png
📸 Capture 04 — Build Docker
https://./captures/04-docker-build.png

Nom de la capture :

text
04-docker-build.png
7. Push de l'image sur Docker Hub
🎯 Objectif
Publier l'image Docker sur Docker Hub pour qu'elle soit accessible au cluster Kubernetes.

🔐 Connexion à Docker Hub
bash
docker login -u arijbelaid
🏷️ Tag de l'image
bash
docker tag jenkins-demo:1.0.0 arijbelaid/jenkins-demo:1.0.0
🚀 Push de l'image
bash
docker push arijbelaid/jenkins-demo:1.0.0
Résultat attendu
text
1.0.0: digest: sha256:5fec756710490240ab79ee9d3e0ba35eb7b5a76d7577fda54e3c099926d0003 size: 2051
📸 Capture 05 — Docker Login
https://./captures/05-docker-login.png

Nom de la capture :

text
05-docker-login.png
📸 Capture 06 — Tag de l'image
https://./captures/06-docker-tag.png

Nom de la capture :

text
06-docker-tag.png
📸 Capture 07 — Push de l'image
https://./captures/07-docker-push.png

Nom de la capture :

text
07-docker-push.png
8. Déploiement Kubernetes
🎯 Objectif
Déployer l'application sur le cluster Kubernetes et l'exposer via un Service NodePort.

🚀 Test local du conteneur
bash
docker run --detach --name container-jenkins --publish 8080:8080 arijbelaid/jenkins-demo:1.0.0
🧪 Test avec cURL
bash
curl http://localhost:8080
Résultat attendu :

html
<!DOCTYPE html>
<html>
<head>
    <title>Spring Boot HTML Page</title>
...
⚙️ Application du Deployment
bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
Résultat attendu
text
deployment.apps/spring-app-deployment created
service/spring-app-service created
📸 Capture 08 — Docker Run
https://./captures/08-docker-run.png

Nom de la capture :

text
08-docker-run.png
📸 Capture 09 — Test cURL
https://./captures/09-curl-test.png

Nom de la capture :

text
09-curl-test.png
📸 Capture 10 — Déploiement Kubernetes
https://./captures/10-k8s-deployment.png

Nom de la capture :

text
10-k8s-deployment.png
9. Installation de Jenkins
🎯 Objectif
Installer Jenkins en tant que service sur le nœud master pour automatiser les pipelines CI/CD.

☕ Installation de Java
bash
sudo apt update
sudo apt install -y openjdk-17-jdk
🔑 Ajout de la clé GPG Jenkins
bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key | \
sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
📦 Ajout du dépôt Jenkins
bash
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | \
sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
🚀 Installation de Jenkins
bash
sudo apt update
sudo apt install -y jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
✅ Vérification du service
bash
sudo systemctl status jenkins
Résultat attendu :

text
Active: active (running)
🔥 Ouverture du port 8080
bash
sudo ufw allow 8080/tcp
🔑 Récupération du mot de passe initial
bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
🌐 Accès depuis Windows
text
http://192.168.159.15:8080
10. Configuration du job Freestyle freestyle_demo
🎯 Objectif
Créer un job Freestyle nommé freestyle_demo avec un script shell.

📄 Script shell
bash
echo 'Bonjour'
ls -l
echo 'Bonjour DevOps' > test.txt
ls -l
🔧 Étapes dans Jenkins
Nouvel Item → freestyle_demo → Freestyle project → OK

Étapes du build → Ajouter une étape au build → Exécuter un script shell

Coller le script

Enregistrer → Lancer un build

✅ Résultat du build
text
+ echo Bonjour
Bonjour
+ ls -l
total 0
+ echo 'Bonjour DevOps'
+ ls -l
total 4
-rw-r--r-- 1 jenkins jenkins 15 Oct 14 21:37 test.txt
Finished: SUCCESS
📸 Capture 11 — Configuration du job Freestyle
https://./captures/11-freestyle-config.png

Nom de la capture :

text
11-freestyle-config.png
📸 Capture 12 — Tableau de bord Jenkins
https://./captures/12-freestyle-dashboard.png

Nom de la capture :

text
12-freestyle-dashboard.png
📸 Capture 13 — Console Output du build
https://./captures/13-freestyle-build-console.png

Nom de la capture :

text
13-freestyle-build-console.png
📸 Capture 14 — Vérification sur la VM
https://./captures/14-freestyle-verify-vm.png

Nom de la capture :

text
14-freestyle-verify-vm.png
🧪 Vérification sur la VM
bash
sudo ls -l /var/lib/jenkins/workspace/freestyle_demo
sudo cat /var/lib/jenkins/workspace/freestyle_demo/test.txt
Résultat :

text
Bonjour DevOps
11. Exécution du fichier Java Test.java
🎯 Objectif
Créer un fichier Test.java dans le workspace et l'exécuter via Jenkins.

📄 Contenu de Test.java
java
public class Test {
    public static void main(String[] args) {
        System.out.println("Hi Master DevOps");
    }
}
📁 Création du fichier
bash
sudo nano /var/lib/jenkins/workspace/freestyle_demo/Test.java
sudo chown jenkins:jenkins /var/lib/jenkins/workspace/freestyle_demo/Test.java
🔧 Modification du job Jenkins
Ajouter au script shell :

bash
echo '--- Compilation et exécution de Test.java ---'
javac Test.java
java Test
✅ Résultat
text
+ javac Test.java
+ java Test
Hi Master DevOps
Finished: SUCCESS
📸 Capture 15 — Fichier Test.java
https://./captures/15-testjava-nano.png

Nom de la capture :

text
15-testjava-nano.png
📸 Capture 16 — Attribution des droits
https://./captures/16-testjava-chown.png

Nom de la capture :

text
16-testjava-chown.png
📸 Capture 17 — Configuration du job
https://./captures/17-testjava-job-config.png

Nom de la capture :

text
17-testjava-job-config.png
📸 Capture 18 — Console Output
https://./captures/18-testjava-console.png

Nom de la capture :

text
18-testjava-console.png
12. Récupération du script depuis GitHub
🎯 Objectif
Modifier le job pour récupérer le script depuis votre dépôt GitHub.

🔗 Dépôt GitHub jenkins-scripts
👉 https://github.com/arijbelaid/jenkins-scripts

📄 script.sh
bash
#!/bin/bash
echo 'Bonjour depuis GitHub'
ls -l
echo 'Bonjour DevOps' > test.txt
ls -l
echo '--- Compilation et exécution de Test.java ---'
javac Test.java
java Test
🔧 Configuration Git dans Jenkins
Dans Gestion de code source → Git :

Paramètre	Valeur
Repository URL	https://github.com/arijbelaid/jenkins-scripts.git
Branch Specifier	*/main
Puis dans Étapes du build :

bash
chmod +x script.sh
./script.sh
📸 Capture 19 — Fichiers sur GitHub
https://./captures/19-github-script.png

Nom de la capture :

text
19-github-script.png
📸 Capture 20 — Push vers GitHub
https://./captures/20-github-push.png

Nom de la capture :

text
20-github-push.png
13. Déclenchement automatique (H/5)
🎯 Objectif
Automatiser le job :

Déclenchement toutes les 5 minutes

Conservation des 2 derniers builds

Suppression du workspace avant chaque build

🔧 Configuration
Dans la section Déclencheurs de build :

Paramètre	Valeur
Construire périodiquement	✅ Coché
Planning	H/5 * * * *
Dans la section Général :

Paramètre	Valeur
Supprimer les anciens builds	✅ Coché
Stratégie	Rotation des journaux
Nombre maximum de builds	2
Dans la section Environment :

Paramètre	Valeur
Delete workspace before build starts	✅ Coché
📸 Capture 21 — Configuration du trigger
https://./captures/21-trigger-config.png

Nom de la capture :

text
21-trigger-config.png
14. Préparation de Jenkins pour Kubernetes
🎯 Objectif
Donner à l'utilisateur jenkins les droits pour utiliser Docker et kubectl.

👤 Ajouter jenkins au groupe docker
bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
🔐 Copier le kubeconfig pour jenkins
bash
sudo mkdir -p /var/lib/jenkins/.kube
sudo cp /home/arij/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
sudo chmod 600 /var/lib/jenkins/.kube/config
✅ Vérification des droits
bash
sudo -u jenkins docker ps
sudo -u jenkins kubectl get nodes
Résultat attendu :

text
NAME      STATUS   ROLES           AGE   VERSION
master    Ready    control-plane   14h   v1.31.14
worker1   Ready    <none>          14h   v1.31.14
worker2   Ready    <none>          14h   v1.31.14
📸 Capture 22 — Ajout au groupe docker
https://./captures/22-jenkins-docker-group.png

Nom de la capture :

text
22-jenkins-docker-group.png
📸 Capture 23 — Copie du kubeconfig
https://./captures/23-kubeconfig-copy.png

Nom de la capture :

text
23-kubeconfig-copy.png
📸 Capture 24 — Vérification des droits
https://./captures/24-jenkins-permissions.png

Nom de la capture :

text
24-jenkins-permissions.png
15. Challenge — Pipeline complet deploy_spring_k8s
🎯 Objectif
Créer un job Freestyle qui automatise tout le pipeline :

Build Maven

Correction du Dockerfile

Build Docker

Push Docker Hub

Déploiement Kubernetes

Vérification

📄 Script complet
bash
#!/bin/bash
set -e

echo "=== 1. BUILD MAVEN ==="
mvn clean install -DskipTests

echo "=== 2. CORRECTION DU DOCKERFILE ==="
sed -i 's|openjdk:17-jdk-slim|eclipse-temurin:17-jre|g' Dockerfile
sed -i 's|openjdk:17-jdk|eclipse-temurin:17-jre|g' Dockerfile
sed -i 's|openjdk:17|eclipse-temurin:17-jre|g' Dockerfile
cat Dockerfile

echo "=== 3. BUILD DOCKER ==="
docker build -t arijbelaid/jenkins-demo:latest .

echo "=== 4. LOGIN DOCKER HUB ==="
echo "$DOCKER_PASS" | docker login -u arijbelaid --password-stdin

echo "=== 5. PUSH DOCKER HUB ==="
docker push arijbelaid/jenkins-demo:latest

echo "=== 6. DÉPLOIEMENT KUBERNETES ==="
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

echo "=== 7. REDÉMARRAGE DES PODS ==="
kubectl rollout restart deployment/spring-app-deployment

echo "=== 8. VÉRIFICATION ==="
kubectl get pods
kubectl get svc
📸 Capture 25 — Build du job
https://./captures/25-deploy-k8s-build.png

Nom de la capture :

text
25-deploy-k8s-build.png
📸 Capture 26 — Console Output du déploiement
https://./captures/26-deploy-k8s-console.png

Nom de la capture :

text
26-deploy-k8s-console.png
16. Vérification finale
🎯 Objectif
Vérifier le bon fonctionnement du pipeline complet.

📊 Pods Kubernetes
bash
kubectl get pods
Résultat :

text
NAME                                     READY   STATUS    RESTARTS   AGE
spring-app-deployment-7859c4b57f-6gcq6   1/1     Running   0          97s
spring-app-deployment-7859c4b57f-vmql    1/1     Running   0          83s
🌐 Service NodePort
bash
kubectl get svc spring-app-service
Résultat :

text
NAME                 TYPE        CLUSTER-IP     PORT(S)          AGE
spring-app-service   NodePort    10.102.25.36   2222:30263/TCP   14h
🌍 Accès à l'application
text
http://192.168.159.15:30263
Message affiché :

Welcome to Jenkins Tutorial for Master DevOps and Cloud!

Welcome to Jenkins CI/CD Pipeline for Spring Boot Microservice application deployed to Kubernetes using Helm

Docker - Minikube - Kubectl - Maven - SonarQube - Trivy - ArgoCD - Slack - Webhook - ngRok - Jenkins - Helm

📸 Capture 27 — Pods en Running
https://./captures/27-k8s-pods-running.png

Nom de la capture :

text
27-k8s-pods-running.png
📸 Capture 28 — Application finale
https://./captures/28-application-test.png

Nom de la capture :

text
28-application-test.png
🏆 Pipeline CI/CD complet
text
📦 Pipeline Jenkins CI/CD
│
├── 🟢 Stage 1 : Clone Repo               (récupère dépôt Git)
├── 🟢 Stage 2 : Build Maven              (mvn clean install)
├── 🟢 Stage 3 : Fix Dockerfile           (openjdk → eclipse-temurin)
├── 🟢 Stage 4 : Build Docker Image       (docker build)
├── 🟢 Stage 5 : Login Docker Hub         (docker login)
├── 🟢 Stage 6 : Push Docker Hub          (docker push)
├── 🟢 Stage 7 : Deploy Kubernetes        (kubectl apply)
├── 🟢 Stage 8 : Restart Pods             (kubectl rollout restart)
├── 🟢 Stage 9 : Verify Pods              (kubectl get pods)
└── 🟢 Stage 10 : Get NodePort            (récupération du port)
🧰 Technologies utilisées
Versionnement
Git

GitHub

SSH

DevOps / CI-CD
Ubuntu Server

Jenkins

Maven

Docker

Kubernetes

containerd

Flannel

Application
Java 17

Spring Boot 2.1.5

📁 Structure du projet
text
jenkins-scripts/
│
├── script.sh                          # Script du Step 3
├── Test.java                          # Fichier Java du Step 2
├── README.md
│
└── captures/                          # 28 captures d'écran
    ├── 01-clone-repo.png
    ├── 02-dockerfile.png
    ├── 03-mvn-build-success.png
    ├── 04-docker-build.png
    ├── 05-docker-login.png
    ├── 06-docker-tag.png
    ├── 07-docker-push.png
    ├── 08-docker-run.png
    ├── 09-curl-test.png
    ├── 10-k8s-deployment.png
    ├── 11-freestyle-config.png
    ├── 12-freestyle-dashboard.png
    ├── 13-freestyle-build-console.png
    ├── 14-freestyle-verify-vm.png
    ├── 15-testjava-nano.png
    ├── 16-testjava-chown.png
    ├── 17-testjava-job-config.png
    ├── 18-testjava-console.png
    ├── 19-github-script.png
    ├── 20-github-push.png
    ├── 21-trigger-config.png
    ├── 22-jenkins-docker-group.png
    ├── 23-kubeconfig-copy.png
    ├── 24-jenkins-permissions.png
    ├── 25-deploy-k8s-build.png
    ├── 26-deploy-k8s-console.png
    ├── 27-k8s-pods-running.png
    └── 28-application-test.png
🌐 Liens
🔗 Dépôt GitHub
👉 https://github.com/arijbelaid/jenkins-scripts

🐳 Docker Hub
👉 https://hub.docker.com/u/arijbelaid

👩‍💻 Auteur
Arij Belaid

Étudiante en Master Pro DevOps & Cloud

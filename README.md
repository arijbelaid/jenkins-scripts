# 🚀 Jenkins CI/CD Pipeline - Lab 1

![Jenkins](https://img.shields.io/badge/Jenkins-2.580.1-red?logo=jenkins)
![Docker](https://img.shields.io/badge/Docker-29.1.3-blue?logo=docker)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.31.14-blue?logo=kubernetes)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.1.5-green?logo=spring)
![Maven](https://img.shields.io/badge/Maven-3.9.12-orange?logo=apache-maven)
![Ubuntu](https://img.shields.io/badge/Ubuntu-26.04-orange?logo=ubuntu)

> **Rapport de TP - Lab 1** : Mise en place d'un pipeline CI/CD complet avec Jenkins, Docker et Kubernetes.

---

## 📋 Table des matières

- [Présentation](#-présentation)
- [Architecture](#-architecture)
- [Prérequis](#-prérequis)
- [Partie I - Workflow manuel](#-partie-i---workflow-manuel-sans-jenkins)
- [Partie II - Workflow Jenkins Freestyle](#-partie-ii---workflow-jenkins-freestyle)
- [Challenge - Déploiement automatisé sur Kubernetes](#-challenge---déploiement-automatisé-sur-kubernetes)
- [Résultats](#-résultats)
- [Auteur](#-auteur)

---

## 🎯 Présentation

Ce projet démontre la mise en place d'un **pipeline CI/CD complet** pour une application **Spring Boot** en utilisant :

- **Jenkins** : Automatisation du pipeline
- **Maven** : Build de l'application Java
- **Docker** : Conteneurisation de l'application
- **Kubernetes** : Orchestration et déploiement (cluster 3 nœuds)
- **Docker Hub** : Registre d'images Docker
- **GitHub** : Hébergement du code source et des scripts

---

## 🏗️ Architecture
┌─────────────────────────────────────────────────────────────────┐
│ CLUSTER KUBERNETES │
│ │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │
│ │ MASTER │ │ WORKER 1 │ │ WORKER 2 │ │
│ │192.168.159.15│ │192.168.159.16│ │192.168.159.17│ │
│ │ │ │ │ │ │ │
│ │ - Jenkins │ │ - Pods │ │ - Pods │ │
│ │ - K8s API │ │ - Flannel │ │ - Flannel │ │
│ │ - kubectl │ │ │ │ │ │
│ └──────────────┘ └──────────────┘ └──────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────┘
▲
│
┌────────┴────────┐
│ DOCKER HUB │
│ arijbelaid/ │
│ jenkins-demo │
└─────────────────┘

text

---

## ⚙️ Prérequis

| Outil | Version | Rôle |
|-------|---------|------|
| Ubuntu Server | 26.04 | Système d'exploitation |
| Java JDK | 17 | Runtime Java |
| Maven | 3.9.12 | Build Java |
| Docker | 29.1.3 | Conteneurisation |
| Kubernetes | 1.31.14 | Orchestration |
| Jenkins | 2.580.1 | Automatisation CI/CD |
| containerd | 2.2.2 | Runtime de conteneurs K8s |
| Flannel | Latest | Réseau CNI Kubernetes |

---

## 🔨 Partie I - Workflow manuel (sans Jenkins)

### Étape 1 : Créer le Dockerfile

![Clone Repo](captures/01-clone-repo.png)
![Dockerfile](captures/02-dockerfile.png)

```bash
git clone https://github.com/wahid007/Jenkins-Demo.git
cd Jenkins-Demo
Dockerfile créé :

dockerfile
FROM eclipse-temurin:17-jre
EXPOSE 8080
WORKDIR /app
COPY target/*.jar app.jar
CMD ["java", "-jar", "app.jar"]
Étape 2 : Build Maven et Docker
https://captures/03-mvn-build-success.png
https://captures/04-docker-build.png

bash
mvn clean install
docker build -t jenkins-demo:1.0.0 .
Résultat Maven :

text
[INFO] BUILD SUCCESS
[INFO] Total time:  01:35 min
Étape 3 : Push sur Docker Hub
https://captures/05-docker-login.png
https://captures/06-docker-tag.png
https://captures/07-docker-push.png

bash
docker login -u arijbelaid
docker tag jenkins-demo:1.0.0 arijbelaid/jenkins-demo:1.0.0
docker push arijbelaid/jenkins-demo:1.0.0
Étape 4 : Déploiement Kubernetes
https://captures/08-docker-run.png
https://captures/09-curl-test.png
https://captures/10-k8s-deployment.png

bash
docker run --detach --name container-jenkins --publish 8080:8080 arijbelaid/jenkins-demo:1.0.0
curl http://localhost:8080

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
🔄 Partie II - Workflow Jenkins Freestyle
Step 1 : Job freestyle_demo
https://captures/11-freestyle-config.png
https://captures/12-freestyle-dashboard.png
https://captures/13-freestyle-build-console.png
https://captures/14-freestyle-verify-vm.png

Script shell :

bash
echo 'Bonjour'
ls -l
echo 'Bonjour DevOps' > test.txt
ls -l
Résultat :

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
Vérification sur la VM :

bash
sudo ls -l /var/lib/jenkins/workspace/freestyle_demo
sudo cat /var/lib/jenkins/workspace/freestyle_demo/test.txt
# Résultat : Bonjour DevOps
Step 2 : Fichier Java Test.java
https://captures/15-testjava-nano.png
https://captures/16-testjava-chown.png
https://captures/17-testjava-job-config.png
https://captures/18-testjava-console.png

Test.java :

java
public class Test {
    public static void main(String[] args) {
        System.out.println("Hi Master DevOps");
    }
}
Résultat Console Output :

text
+ javac Test.java
+ java Test
Hi Master DevOps
Finished: SUCCESS
Step 3 : Récupération depuis GitHub
https://captures/19-github-script.png
https://captures/20-github-push.png

script.sh :

bash
#!/bin/bash
echo 'Bonjour depuis GitHub'
ls -l
echo 'Bonjour DevOps' > test.txt
ls -l
echo '--- Compilation et exécution de Test.java ---'
javac Test.java
java Test
Step 4 : Déclenchement automatique
https://captures/21-trigger-config.png

Configuration :

Déclencheur : H/5 * * * * (toutes les 5 minutes)

Conservation : 2 derniers builds

Nettoyage : Workspace supprimé avant chaque build

🏆 Challenge - Déploiement automatisé sur Kubernetes
Préparation de Jenkins
https://captures/22-jenkins-docker-group.png
https://captures/23-kubeconfig-copy.png
https://captures/24-jenkins-permissions.png

bash
sudo usermod -aG docker jenkins
sudo mkdir -p /var/lib/jenkins/.kube
sudo cp /home/arij/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
sudo chmod 600 /var/lib/jenkins/.kube/config
sudo systemctl restart jenkins

# Vérification
sudo -u jenkins docker ps
sudo -u jenkins kubectl get nodes
Résultat :

text
NAME      STATUS   ROLES           AGE   VERSION
master    Ready    control-plane   14h   v1.31.14
worker1   Ready    <none>          14h   v1.31.14
worker2   Ready    <none>          14h   v1.31.14
Job deploy_spring_k8s
https://captures/25-deploy-k8s-build.png
https://captures/26-deploy-k8s-console.png

Script complet du pipeline :

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

echo "=== 6. CRÉATION DES FICHIERS YAML ==="
cat > deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app-container
        image: arijbelaid/jenkins-demo:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 2222
EOF

cat > service.yaml <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: spring-app-service
spec:
  selector:
    app: spring-app
  ports:
  - protocol: TCP
    port: 2222
    targetPort: 2222
  type: NodePort
EOF

echo "=== 7. DÉPLOIEMENT KUBERNETES ==="
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

echo "=== 8. REDÉMARRAGE DES PODS ==="
kubectl rollout restart deployment/spring-app-deployment

echo "=== 9. VÉRIFICATION ==="
kubectl get pods
kubectl get svc

echo "=== 10. RÉCUPÉRATION DU NODEPORT ==="
NODEPORT=$(kubectl get svc spring-app-service -o jsonpath='{.spec.ports[0].nodePort}')
echo "NodePort : $NODEPORT"
echo "URL : http://192.168.159.15:$NODEPORT"

echo "DÉPLOIEMENT TERMINÉ AVEC SUCCÈS !"
Résultats du déploiement
https://captures/27-k8s-pods-running.png
https://captures/28-application-test.png

text
=== 9. VÉRIFICATION ===
NAME                                     READY   STATUS    RESTARTS   AGE
spring-app-deployment-7859c4b57f-6gcq6   1/1     Running   0          97s
spring-app-deployment-7859c4b57f-vmql    1/1     Running   0          83s

NAME                 TYPE        CLUSTER-IP     PORT(S)          AGE
spring-app-service   NodePort    10.102.25.36   2222:30263/TCP   14h

=== 10. RÉCUPÉRATION DU NODEPORT ===
NodePort : 30263
URL : http://192.168.159.15:30263

DÉPLOIEMENT TERMINÉ AVEC SUCCÈS !
Finished: SUCCESS
📊 Résultats
Application déployée
L'application Spring Boot est accessible à l'adresse :
http://192.168.159.15:30263

Message affiché :

Welcome to Jenkins Tutorial for Master DevOps and Cloud!

Welcome to Jenkins CI/CD Pipeline for Spring Boot Microservice application deployed to Kubernetes using Helm

Docker - Minikube - Kubectl - Maven - SonarQube - Trivy - ArgoCD - Slack - Webhook - ngRok - Jenkins - Helm

Bilan du TP
Étape	Description	Statut
Partie I	Workflow manuel (Docker + K8s)	✅
Step 1	Job Freestyle freestyle_demo	✅
Step 2	Exécution Java Test.java	✅
Step 3	Récupération script depuis GitHub	✅
Step 4	Déclenchement auto + nettoyage	✅
Challenge	Pipeline complet build → K8s	✅
Cluster K8s	1 master + 2 workers	✅
Image Docker	Publiée sur Docker Hub	✅
📚 Technologies utilisées
Technologie	Utilisation
Jenkins	Automatisation du pipeline CI/CD
Maven	Compilation de l'application Spring Boot
Docker	Conteneurisation de l'application
Kubernetes	Orchestration des conteneurs
Flannel	Réseau CNI pour Kubernetes
containerd	Runtime de conteneurs Kubernetes
Docker Hub	Registre d'images Docker
GitHub	Hébergement du code source
📁 Structure du projet
text
jenkins-scripts/
├── captures/
│   ├── 01-clone-repo.png
│   ├── 02-dockerfile.png
│   ├── 03-mvn-build-success.png
│   ├── 04-docker-build.png
│   ├── 05-docker-login.png
│   ├── 06-docker-tag.png
│   ├── 07-docker-push.png
│   ├── 08-docker-run.png
│   ├── 09-curl-test.png
│   ├── 10-k8s-deployment.png
│   ├── 11-freestyle-config.png
│   ├── 12-freestyle-dashboard.png
│   ├── 13-freestyle-build-console.png
│   ├── 14-freestyle-verify-vm.png
│   ├── 15-testjava-nano.png
│   ├── 16-testjava-chown.png
│   ├── 17-testjava-job-config.png
│   ├── 18-testjava-console.png
│   ├── 19-github-script.png
│   ├── 20-github-push.png
│   ├── 21-trigger-config.png
│   ├── 22-jenkins-docker-group.png
│   ├── 23-kubeconfig-copy.png
│   ├── 24-jenkins-permissions.png
│   ├── 25-deploy-k8s-build.png
│   ├── 26-deploy-k8s-console.png
│   ├── 27-k8s-pods-running.png
│   └── 28-application-test.png
├── script.sh
├── Test.java
└── README.md
👤 Auteur
Arij Belaid

🎓 Étudiante en Master DevOps & Cloud

🐳 Docker Hub : arijbelaid

🐙 GitHub : @arijbelaid

📝 Remerciements
Enseignant : Wahid Hamdi

Dépôt source : wahid007/Jenkins-Demo


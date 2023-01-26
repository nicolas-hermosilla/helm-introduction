# Utilisation de Helm

# Installer Helm 
> Via Package Manager  
https://helm.sh/docs/intro/install/

```bash=
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

# Déployer un nextcloud en utilisant la chart helm officielle

https://artifacthub.io/packages/helm/nextcloud/nextcloud

Nexcloud est un serveur de partage de fichiers qui remet le contrôle et la sécurité de vos propres données entre vos mains.

## Déployer un cluster kind
```bash=
kind create cluster
```

## Ajouter et déployer nextcloud avec Helm Chart
```bash=
helm repo add nextcloud https://nextcloud.github.io/helm/
> "nextcloud" has been added to your repositories

helm install my-release nextcloud/nextcloud

NAME: my-release
LAST DEPLOYED: Thu Jan 26 09:11:56 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
#######################################################################################################
## WARNING: You did not provide an external database host in your 'helm install' call                ##
## Running Nextcloud with the integrated sqlite database is not recommended for production instances ##
#######################################################################################################

For better performance etc. you have to configure nextcloud with a resolvable database
host. To configure nextcloud to use and external database host:


1. Complete your nextcloud deployment by running:

  export APP_HOST=127.0.0.1
  export APP_PASSWORD=$(kubectl get secret --namespace default my-release-nextcloud -o jsonpath="{.data.nextcloud-password}" | base64 --decode)

  ## PLEASE UPDATE THE EXTERNAL DATABASE CONNECTION PARAMETERS IN THE FOLLOWING COMMAND AS NEEDED ##

  helm upgrade my-release nextcloud/nextcloud \
    --set nextcloud.password=$APP_PASSWORD,nextcloud.host=$APP_HOST,service.type=ClusterIP,mariadb.enabled=false,externalDatabase.user=nextcloud,externalDatabase.database=nextcloud,externalDatabase.host=YOUR_EXTERNAL_DATABASE_HOST

echo $APP_PASSWORD
> changeme
```
```bash=
$ helm upgrade my-release nextcloud/nextcloud \
    --set nextcloud.password=$APP_PASSWORD,nextcloud.host=$APP_HOST,service.type=ClusterIP,mariadb.enabled=false,externalDatabase.user=nextcloud,externalDatabase.database=nextcloud,externalDatabase.host=YOUR_EXTERNAL_DATABASE_HOST
    
Release "my-release" has been upgraded. Happy Helming!
NAME: my-release
LAST DEPLOYED: Thu Jan 26 09:15:46 2023
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
1. Get the nextcloud URL by running:

  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=nextcloud" -o jsonpath="{.items[0].metadata.name}")
  echo http://127.0.0.1:8080/
  kubectl port-forward $POD_NAME 8080:80

2. Get your nextcloud login credentials by running:

  echo User:     admin
  echo Password: $(kubectl get secret --namespace default my-release-nextcloud -o jsonpath="{.data.nextcloud-password}" | base64 --decode)
```

## Se connecter a Nextcloud

```
$ kubectl port-forward $POD_NAME 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
```

![](https://i.imgur.com/O3vyMHM.png)




# Créer un fichier values.yaml pour augmenter le nombre de replicas et déployer la mise à jour
* Créer le fichier values en indiquant le nombre de réplicas voulu
```
vim values.yml
replicaCount: 3
```
* Updater le chart avec le fichier values.yml
    * Exectuer une nouvelle fois la commande helm upgrade en précisant le fichier values.yml
```
helm upgrade my-release nextcloud/nextcloud --set nextcloud.password=$APP_PASSWORD,nextcloud.host=$APP_HOST,service.type=ClusterIP,mariadb.enabled=false,externalDatabase.user=nextcloud,externalDatabase.database=nextcloud,externalDatabase.host=YOUR_EXTERNAL_DATABASE_HOST -f values.yml
```
* Vérifier que les pods sont bien créés
```
kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
my-release-nextcloud-8c65698f9-jzlmj   1/1     Running   0          48m
my-release-nextcloud-8c65698f9-lg9zf   1/1     Running   0          48m
my-release-nextcloud-8c65698f9-n692d   1/1     Running   0          48m
```

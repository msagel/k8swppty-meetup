# k8swppty-meetup

## Requerimientos para levantar Wordpress en K8s

- [Minikube][mkinst], [microk8s][mk8s], [docker for windows][dw] con k8s habilitado, [docker for mac][dm] con k8s habilitado o algun cluster en la nube ([AKS][aks], [GKE][gke], [EKS][eks])
- Cliente de [kubectl][kubectl]
- Cliente de [Helm][helm]

## Repositorio de HELM

Usaremos el repositorio de [helm de bitnami][bitwp] y ademas el repo de [stable de nfs-server][nfs] para poder tener mas de una replica de wordpress en kubernetes.

## Agregar repo a helm y actualizar

```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com
$ helm repo update
```

## Despliegue de Wordpress

Crearemos un wordpress altamente reduntante que contendra 2 base de datos replicadas, para poder aumentar replicas tendriamos que hacer que wp-content este en un compartido nsf que puedan accesar mas de un contenedor. El comando para levantar la solucion en una solucion de nube es el siguiente:

```bash
$ helm install nfs-server stable/nfs-server-provisioner --set persistence.enabled=true,persistence.size=60Gi
$ helm install k8swp bitnami/wordpress --set mariadb.replication.enabled=true,persistence.accessMode=ReadWriteMany,persistence.storageClass=nfs,mariadb.master.persistence.storageClass=nfs
```

## Prueba de servicios

```bash
kubectl get svc --namespace default -w k8swp-wordpress
```

## Aumentar las replicas

```bash
helm install k8swp bitnami/wordpress --set replicaCount=3,mariadb.replication.enabled=true,persistence.accessMode=ReadWriteMany,persistence.storageClass=nfs,mariadb.master.persistence.storageClass=nfs,mariadb.master.persistence.accessModes=ReadWriteMany
```

---

[mkinst]: https://github.com/kubernetes/minikube
[mk8s]: https://microk8s.io/
[dw]: https://docs.docker.com/docker-for-windows/install/
[dm]: https://docs.docker.com/docker-for-mac/install/
[aks]: https://azure.microsoft.com/en-us/services/kubernetes-service/
[gke]: https://cloud.google.com/kubernetes-engine
[eks]: https://aws.amazon.com/es/eks/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc&eks-blogs.sort-by=item.additionalFields.createdDate&eks-blogs.sort-order=desc
[kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[helm]: https://helm.sh/docs/intro/install/
[bitwp]: https://github.com/bitnami/charts/tree/master/bitnami/wordpress
[nfs]: https://github.com/helm/charts/tree/master/stable/nfs-server-provisioner

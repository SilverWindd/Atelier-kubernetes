# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :** Ladeira Evan
**Groupe :** RS-2

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > 3 noeuds s'affichent : `k3d-lab-server-0` qui joue le rôle de cerveau du cluster (control-plane) : il héberge le plan de contrôle Kubernetes (API Server, etcd, controller-manager, scheduler) qui gère l'état global et pilote le cluster. Ensuite, il y a `k3d-lab-agent-0` et `k3d-lab-agent-1` qui sont les workers : ils exécutent les charges de travail.
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   > `docker ps` montre les conteneurs Docker en cours d'exécution sur la machine hôte. Dans k3d, un « nœud » est un conteneur Docker qui fait tourner une instance de kubernetes. Il faut cependant distinguer les 3 nœuds Kubernetes des 4 conteneurs Docker : le Load Balancer est un conteneur supplémentaire, mais ce n'est pas un nœud Kubernetes.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > La commande `kubectl create deployment` (suivie de `kubectl expose`) a généré trois objets principaux (visibles dans kubectl get all). Tout d'abord, un Deployment (deployment.apps/web) : il gère le déploiement de l'application et garantit le nombre souhaité de Pods. Ensuite, un ReplicaSet (replicaset.apps/web-68d995574f) : il garantit qu'un Pod correspondant au nombre de réplicas demandé est toujours disponible. Enfin, un Pod (pod/web-68d995574f-v54gz) : il contient le conteneur qui exécute l'application Nginx.   
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Kubernetes a automatiquement créé un nouveau Pod pour le remplacer. le Deployment demande qu'un seul Pod soit toujours actif. Le ReplicaSet surveille le nombre de Pods disponibles et en recrée automatiquement un lorsqu'il en manque un.
   
## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non, le message « Bienvenue sur mon site Web » n'a pas survécu à la suppression du Pod. En effet, lorsqu'un Pod est supprimé, son conteneur et les modifications effectuées dans son système de fichiers sont perdus. Le nouveau Pod créé par Kubernetes repart de l'image originale nginx, qui contient sa propre page d'accueil.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Le port-forward permet de rediriger le trafic d'un port local de la machine (ici 8080) vers un port du Service Kubernetes (80). Il a cessé de fonctionner parce que la connexion était liée au Pod qui avait été supprimé. Le port-forward est une connexion temporaire.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > `kubectl create` : Crée une ressource à partir d'un manifeste, si elle existe déjà dans le cluster, la commande échoue.
   > `kubectl appluy` : crée la ressource si elle n'existe pas, ou alors la met à jour si elle existe.
     
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Il permet au Service de cibler dynamiquement les pods vers lesquels router le trafic en filtrant sur leurs labels. S'il n'y a aucune correspondance, le Service est bien créé, mais ses endpoints restent vides. Aucun trafic ne peut être acheminé et les requêtes vers l'adresse du Service échouent, car le cluster ne trouve aucun pod correspondant à l'étiquette.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
@SilverWindd ➜ /workspaces/Atelier-kubernetes (main) $ kubectl get pods -l app=site -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP          NODE               NOMINATED NODE   READINESS GATES
site-645899988f-4lb9f   1/1     Running   0          15m   10.42.0.7   k3d-lab-agent-0    <none>           <none>
site-645899988f-xpt2b   1/1     Running   0          15m   10.42.2.7   k3d-lab-server-0   <none>           <none>
site-645899988f-zz9fc   1/1     Running   0          15m   10.42.1.8   k3d-lab-agent-1    <none>           <none>

10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Il avait le statut ErrImagePull puis ImagePullBackOff

    @SilverWindd ➜ /workspaces/Atelier-kubernetes (main) $ kubectl describe pod site-5f7f944589-knr9j
[...]
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  60s                default-scheduler  Successfully assigned default/site-5f7f944589-knr9j to k3d-lab-agent-0
  Normal   Pulling    20s (x3 over 60s)  kubelet            spec.containers{nginx}: Pulling image "nginx:9.99"
  Warning  Failed     20s (x3 over 59s)  kubelet            spec.containers{nginx}: Failed to pull image "nginx:9.99": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:9.99": failed to resolve reference "docker.io/library/nginx:9.99": docker.io/library/nginx:9.99: not found
  Warning  Failed     20s (x3 over 59s)  kubelet            spec.containers{nginx}: Error: ErrImagePull
  Normal   BackOff    4s (x3 over 59s)   kubelet            spec.containers{nginx}: Back-off pulling image "nginx:9.99"
  Warning  Failed     4s (x3 over 59s)   kubelet            spec.containers{nginx}: Error: ImagePullBackOff
  
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui, le site restait 100 % accessible grâce au RollingUpdate du Deployment. Kubernetes ne supprime les anciens pods fonctionnels (nginx:1.28) qu'une fois que les nouveaux pods sont démarrés et prêts (Ready). Comme le nouveau pod nginx:9.99 a échoué au téléchargement de l'image, le déploiement s'est bloqué, conservant les anciens pods actifs pour continuer à servir le trafic via le Service.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://miniature-halibut-7479945vpxxhpx5w-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > Oui, Portainer et kubectl communiquent exactement avec le même composant : l'API Server de Kubernetes. Portainer n'est pas un système de déploiement parallèle : il fonctionne comme une interface graphique.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    > A : Le site est directement accessible sur le port 8081 sans avoir à utiliser de port forwarding.
    > B : Si la probe renvoie un chemin invalide, Nginx va renvoyer une erreur 404 Not Found pour cette URL. La sonde considérera que le test a échoué. Par conséquent, Kubernetes marquera le pod comme non prêt (Ready: 0/1) et retirera immédiatement le pod des endpoints du Service : le site deviendra inaccessible.
    > C : Une fois le uncordon effectué, le nœud est de nouveau prêt à accepter de nouveaux pods, mais les pods qui ont été déplacés ne reviennent pas automatiquement sur agent-0. Kubernetes maintient l'état équilibré actuel. Pour forcer leur redistribution, il faudrait redémarrer le déploiement (kubectl rollout restart deploy/site).

---

@SilverWindd ➜ /workspaces/Atelier-kubernetes/scripts (main) $ ./check.sh 

Exercice 1 — Cluster
  ✔ Le cluster k3d 'lab' existe
  ✔ 3 nœuds sont Ready (1 server + 2 agents)
  ✔ Le port 8081 est relié au load balancer

Exercice 2 — Premier déploiement
  ✔ Le Deployment 'web' est disponible
  ✔ Le Service 'web' existe

Exercice 4 — Déclaratif
  ✔ Les TODO des manifestes k8s/site/ sont complétés
  ✔ Le Deployment 'site' est disponible
  ✔ 'site' utilise la ConfigMap 'site-html'
  ✔ Le Service 'site' a des endpoints

Exercice 5 — Scaling & mises à jour
  ✔ 'site' a au moins 3 réplicas prêts
  ✔ Les pods 'site' sont répartis sur ≥ 2 nœuds
  ✔ L'image de 'site' est versionnée (nginx:x.y)
  ✔ Au moins une mise à jour a été déployée

Exercice 6 — Portainer & Mario
  ✔ Portainer est disponible
  ✔ Le Deployment 'mario' est disponible

Bonus
  ✔ A — L'Ingress 'site' répond sur le port 8081
  ✔ B — 'site' a une readinessProbe
  ✔ B — 'site' a des limites de ressources

Score : 18 / 18  (l'exercice 3 n'est pas vérifiable automatiquement)

**Score obtenu avec `./scripts/check.sh` :** 18 / 18

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
   >
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   ```

   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    >
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    >

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    >

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** ... / ...

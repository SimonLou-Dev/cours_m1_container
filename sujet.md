# Projet — Déployer phpMyAdmin + MySQL sur Kubernetes

**ESGI · Cours VXLAN & Kubernetes**

- **Format** : individuel (solo)
- **Rendu** : fin du J3, avant 17h00
- **Poids dans la note** : 1/2 (l'autre moitié = quiz de 20 questions)

---

## Contexte

Vous êtes ops sur un nouveau cluster Kubernetes (kind, 3 nœuds, sur votre poste).

Vous devez **mettre en production** une instance phpMyAdmin connectée à un MySQL.

- **MySQL 8** : base de données, doit survivre à un redémarrage de pod
- **phpMyAdmin** : interface web, configurée en `PMA_ARBITRARY=1` pour ne pas avoir à fixer un host (zéro process d'install)
- Tout doit être **déclaré en YAML**, idempotent, ré-appliquable

---

## Contraintes techniques

1. **Cluster kind** à 3 nœuds minimum (1 control-plane + 2 workers)
2. **Aucun mot de passe en clair** dans les YAML (utiliser un `Secret`)
3. La configuration de phpMyAdmin doit venir d'une `ConfigMap`
4. MySQL doit avoir un **volume persistant** : la donnée ne doit pas disparaître à `delete pod`
5. phpMyAdmin doit être **scalable** (replicas ≥ 2), exposé via NodePort ou Ingress
6. Une **NetworkPolicy** doit interdire l'accès à MySQL depuis tout Pod *autre que* phpMyAdmin
7. **Kite** doit être installé et accessible (GUI cluster — remplace l'ancien Dashboard archivé)
8. Toutes les ressources doivent avoir des **labels cohérents** (`app=`, `tier=`)

---

## Démarrage rapide — fichier kind

```yaml
# kind-cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - { containerPort: 30080, hostPort: 8080, protocol: TCP }   # phpMyAdmin
      - { containerPort: 80,    hostPort: 80,   protocol: TCP }   # Ingress (optionnel)
      - { containerPort: 443,   hostPort: 443,  protocol: TCP }   # Ingress (optionnel)
  - role: worker
  - role: worker
```

```bash
kind create cluster --config kind-cluster.yaml --name esgi
```

---

## Images officielles à utiliser

- `mysql:8` ([Docker Hub](https://hub.docker.com/_/mysql))
- `phpmyadmin:latest` ([Docker Hub](https://hub.docker.com/_/phpmyadmin))

Variables d'env utiles :
- MySQL : `MYSQL_ROOT_PASSWORD` (obligatoire)
- phpMyAdmin : `PMA_ARBITRARY=1` (laisse choisir l'hôte au login → on tape `mysql` qui résoudra via DNS interne)

→ **Aucun code applicatif à écrire**, juste du YAML.

---

## Test de votre projet

À la fin, vous devez pouvoir :

1. `kind delete cluster --name esgi`
2. `kind create cluster --config kind-cluster.yaml --name esgi`
3. `kubectl apply -f .` (depuis votre dossier de YAML)
4. Attendre que tout passe `Running` (`kubectl get pods -w`)
5. Ouvrir http://localhost:8080 → page de login phpMyAdmin
6. Se connecter avec `host=mysql`, `user=root`, `pass=<celui du Secret>`
7. Créer une DB de test, y mettre des données
8. `kubectl delete pod -l app=mysql` → le Pod redémarre, **les données sont toujours là**

Si ça marche → vous avez tout bon.

---

## Livrable

Un dossier (Git de préférence, sinon ZIP) contenant :

- **Tous vos manifests YAML**, organisés (au choix : un fichier par objet, ou regroupés en `mysql.yaml`, `phpmyadmin.yaml`…)
- Le fichier `kind-cluster.yaml`
- Un `README.md` (10-15 lignes) qui explique :
  - Comment créer le cluster
  - Comment déployer
  - Comment tester (URL, login, étapes)
- Un dossier `screenshots/` avec :
  - `01-kubectl-get-all.png` — tout vos objets running
  - `02-phpmyadmin-connected.png` — interface PMA après login
  - `03-persistance.png` — la DB visible avant ET après `delete pod`
  - `04-networkpolicy-deny.png` — un Pod tiers qui n'arrive pas à joindre mysql:3306
  - `05-networkpolicy-allow.png` — phpMyAdmin qui réussit à joindre mysql:3306
  - `06-ha.png` — un `kubectl get pods` avant et après suppression d'un Pod, ou un `docker stop` d'un worker
  - `07-kite.png` — Kite montrant vos objets (Pods, Services, Deployments du projet)

→ Évalué sur la **grille** (voir `grille-evaluation.md`).

---

## Conseils

- **Itérez en petit** : commencez par MySQL seul, validez, puis ajoutez phpMyAdmin, puis Secret, puis NetworkPolicy…
- Utilisez `kubectl explain <objet>.<champ>` quand vous doutez de la syntaxe
- `kubectl get all` est votre meilleur ami
- En cas de blocage : `kubectl describe pod <nom>` puis `kubectl logs <nom>`
- Pas la peine de fignoler la mise en page de Kite, on veut voir **vos** Pods dedans

→ **Le but n'est pas la beauté du YAML, c'est de prouver que vous avez compris les concepts.**

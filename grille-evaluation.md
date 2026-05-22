# Grille d'évaluation — Projet phpMyAdmin + MySQL

**Total : /20 — Compte pour 1/2 de la note finale.**

L'évaluateur vérifie chaque critère soit en lisant les YAML, soit en demandant une démo live, soit via les captures du dossier `screenshots/`.

---

## Infrastructure (3 pts)

| Critère | Pts | Vérification |
|---|---|---|
| Cluster kind à 3 nœuds (1 CP + 2 workers minimum) | 1 | `kubectl get nodes` montre 3 Ready |
| `kind-cluster.yaml` fourni et fonctionnel | 1 | Le prof recrée le cluster avec ce fichier |
| `kubectl apply -f .` déploie tout en une commande | 1 | Test en live |

---

## Base de données (4 pts)

| Critère | Pts | Vérification |
|---|---|---|
| Deployment MySQL fonctionnel | 1 | Pod Running |
| PVC + PersistentVolume monté sur `/var/lib/mysql` | 1 | YAML + capture `03-persistance.png` |
| Données conservées après `kubectl delete pod` | 2 | Démo live OU capture `03-persistance.png` claire |

---

## phpMyAdmin (4 pts)

| Critère | Pts | Vérification |
|---|---|---|
| Deployment phpMyAdmin avec replicas ≥ 2 | 1 | `kubectl get deploy phpmyadmin` |
| Exposé via NodePort **ou** Ingress, accessible navigateur | 2 | Capture `02-phpmyadmin-connected.png` |
| Connexion réelle à MySQL via DNS interne (host=mysql) | 1 | Démontré dans la capture |

---

## Sécurité / Configuration (5 pts)

| Critère | Pts | Vérification |
|---|---|---|
| `Secret` pour le mot de passe MySQL, référencé par `secretKeyRef` | 2 | Aucun mot de passe en clair dans les YAML |
| `ConfigMap` pour la configuration phpMyAdmin (`PMA_ARBITRARY` etc.) | 1 | Lecture YAML |
| `NetworkPolicy` : seul `app=phpmyadmin` peut joindre `app=mysql:3306` | 1 | Captures `04` + `05` |
| Labels cohérents (`app`, `tier`) sur Pods, Services, Deployments | 1 | Lecture YAML |

---

## Production-readiness (3 pts)

| Critère | Pts | Vérification |
|---|---|---|
| Démo HA : delete pod ou stop d'un worker, le service répond toujours | 1 | Capture `06-ha.png` ou démo live |
| Kite installé et accessible (GUI cluster) | 1 | Capture `07-kite.png` |
| README clair : créer cluster → déployer → tester en moins de 5 commandes | 1 | Lecture du README |

---

## Bonus (jusqu'à +2 pts, plafonné à 20)

- **+1** : Ingress avec hostname (`pma.local`) et `extraPortMappings` 80/443
- **+1** : Probes (liveness + readiness) sur les deux Deployments
- **+1** : PodDisruptionBudget pour phpMyAdmin
- **+1** : Tout dans un namespace dédié (`projet-pma`) au lieu de `default`

---

## Pénalités

- **−2** : Mot de passe en clair dans un YAML
- **−1** : Pas de README ou README inutilisable
- **−1** : Pas de captures d'écran
- **−3** : YAML non ré-appliquable (`kubectl apply` plante au 2e passage)

---

## Notation finale

| Note | Niveau |
|---|---|
| 18-20 | Maîtrise complète, prêt pour la prod |
| 14-17 | Tous les concepts vus en cours sont appliqués |
| 10-13 | Les bases sont là, la sécurité est faible |
| 5-9 | Déploiement fonctionnel mais beaucoup de concepts manquent |
| < 5 | Le projet ne fonctionne pas |

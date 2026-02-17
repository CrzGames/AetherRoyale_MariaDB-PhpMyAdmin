# AetherRoyale – MariaDB / phpMyAdmin (Kubernetes)

Déploiement de MariaDB pour les environnements Kubernetes du projet **Aether Royale**.

* Staging : MariaDB standalone (simple, léger, non HA)
* Production : MariaDB HA (à prévoir)
* Administration : phpMyAdmin (interface web sécurisée)

<br /><br />

---

<br /><br />

# 📦 Structure du dépôt

```
AetherRoyale_MariaDB-PhpMyAdmin/
  k8s/
    mariadb/
      staging/
        mariadb.yaml
      production/
        mariadb.yaml
    phpmyadmin/
      staging/
        phpmyadmin.yaml
        phpmyadmin-ingress.yaml
      production/
        phpmyadmin.yaml
        phpmyadmin-ingress.yaml
```

Chaque dossier représente un environnement indépendant.

<br /><br />

---

<br /><br />

# 🧰 Prérequis

* Cluster Kubernetes fonctionnel
* Accès `kubectl` configuré
* cert-manager installé (pour TLS)
* Ingress Controller NGINX installé

<br /><br />

---

<br /><br />

# ⚙️ Configuration

Éditer le fichier correspondant à l’environnement :

```
k8s/mariadb/staging/mariadb.yaml
```

ou

```
k8s/mariadb/production/mariadb.yaml
```

Remplacer uniquement les valeurs sensibles dans le Secret :

```yaml
MARIADB_USER: REPLACE_ME_BASE64
MARIADB_PASSWORD: REPLACE_ME_BASE64
MARIADB_ROOT_PASSWORD: REPLACE_ME_BASE64
```

Ne pas modifier :

```
MARIADB_DATABASE
```

(le nom de la base est fixé par environnement)

<br /><br />

---

<br /><br />

# 🔐 Secrets Kubernetes & Base64 (Important)

Kubernetes demande du **base64 uniquement pour les objets `Secret`** lorsque les valeurs sont écrites directement dans un fichier YAML.

Ce n’est PAS du chiffrement, seulement un encodage.

Exemple :

```
testuser → dGVzdHVzZXI=
```

Encoder une valeur :

Linux / Mac :

```bash
echo -n "monuser" | base64
```

Windows (PowerShell) :

```powershell
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("monuser"))
```

Note importante :

Si tu crées un secret avec :

```bash
kubectl create secret generic ...
```

Kubernetes encode automatiquement en base64 pour toi.

Le base64 est nécessaire uniquement quand tu écris les valeurs directement dans un YAML.

<br /><br />

---

<br /><br />

# 🚀 Installation MariaDB / phpMyAdmin

## 🧪 Installation – Staging

Déployer MariaDB :

```bash
kubectl apply -f k8s/mariadb/staging/mariadb.yaml
```

Déployer phpMyAdmin :

```bash
kubectl apply -f k8s/phpmyadmin/staging/phpmyadmin.yaml
kubectl apply -f k8s/phpmyadmin/staging/phpmyadmin-ingress.yaml
```

## 🏭 Installation – Production

Déployer MariaDB :

```bash
kubectl apply -f k8s/mariadb/production/mariadb.yaml
```

Déployer phpMyAdmin :

```bash
kubectl apply -f k8s/phpmyadmin/production/phpmyadmin.yaml
kubectl apply -f k8s/phpmyadmin/production/phpmyadmin-ingress.yaml
```

<br /><br />

---

<br /><br />

# 📡 Accès MariaDB depuis le cluster

## Staging

Host :

```
mariadb.staging-db.svc.cluster.local
```

Port :

```
3306
```

## Production

Host :

```
mariadb.production-db.svc.cluster.local
```

Port :

```
3306
```

<br /><br />

---

<br /><br />

# 🔐 Variables d’environnement (backend)

## Staging

```
DB_HOST=mariadb.staging-db.svc.cluster.local
DB_PORT=3306
DB_USER=REPLACE_ME
DB_PASSWORD=REPLACE_ME
DB_DATABASE=aetherroyale_staging
```

## Production

```
DB_HOST=mariadb.production-db.svc.cluster.local
DB_PORT=3306
DB_USER=REPLACE_ME
DB_PASSWORD=REPLACE_ME
DB_DATABASE=aetherroyale_production
```

<br /><br />

---

<br /><br />

# 🧠 phpMyAdmin (Interface Web)

phpMyAdmin permet de :

* Visualiser les tables
* Exécuter des requêtes SQL
* Debug les données
* Gérer les utilisateurs

## 🔒 Protection par BasicAuth

Créer le fichier auth :

```bash
sudo apt-get update && sudo apt-get install -y apache2-utils
htpasswd -nbB admin 'MONPASSWORD' > auth
```

Créer le secret Kubernetes :

```bash
# modifier le namespace si besoin entre staging / prod
kubectl -n staging-db create secret generic phpmyadmin-basic-auth --from-file=auth
```

<br /><br />

---

<br /><br />

# Accès phpMyAdmin

## Staging

```
https://staging.phpmyadmin.aetherroyale.crzgames.com
```

## Production

```
https://phpmyadmin.aetherroyale.crzgames.com
```

Un login/mot de passe BasicAuth sera demandé avant l’accès.

<br /><br />

---

<br /><br />

# 🧱 Notes Architecture

## Staging

* MariaDB standalone
* 1 PVC de 10Go
* Pas de haute disponibilité
* 1 seule pod
* phpMyAdmin exposé via Ingress sécurisé

## Production

* Réplication MariaDB
* Sauvegardes automatiques
* Haute disponibilité
* Monitoring

<br /><br />

---

<br /><br />

# 🔒 Bonnes pratiques

* Ne jamais exposer MariaDB publiquement
* Accès uniquement interne au cluster
* Toujours utiliser des mots de passe forts
* Garder phpMyAdmin protégé par BasicAuth
* Utiliser des credentials différents entre staging et production

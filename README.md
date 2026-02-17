# AetherRoyale – MariaDB / phpMyAdmin (Kubernetes)

Déploiement de la base de données MariaDB pour les environnements Kubernetes du projet **Aether Royale**.

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
* Ingress Controller NGINX installé
* cert-manager installé (TLS automatique)

<br /><br />

---

<br /><br />

# ⚙️ Configuration

Éditer le fichier :

```
k8s/mariadb/staging/mariadb.yaml
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

(le nom de la base est volontairement fixé pour l’environnement staging)

<br /><br />

---

<br /><br />

# 🔐 Secrets Kubernetes & Base64 (Important)

Oui, Kubernetes demande du **base64 uniquement pour les objets `Secret`**.

Ce n’est **PAS du chiffrement**, juste un encodage obligatoire dans les fichiers YAML.

Exemple :

```
testuser → dGVzdHVzZXI=
```

### Encoder une valeur

#### Linux / Mac

```bash
echo -n "monuser" | base64
```

#### Windows (PowerShell)

```powershell
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("monuser"))
```

<br />

⚠️ Note importante :

Quand tu fais :

```bash
kubectl create secret generic ...
```

Kubernetes encode automatiquement en base64 pour toi.

Le base64 est nécessaire uniquement quand tu écris les valeurs directement dans un YAML comme ici.

<br /><br />

---

<br /><br />

# 🚀 Installation MariaDB / phpMyAdmin

## 🧪 Installation – Staging

### 1) Déployer MariaDB

```bash
kubectl apply -f k8s/mariadb/staging/mariadb.yaml
```

### 2) Déployer phpMyAdmin

```bash
kubectl apply -f k8s/phpmyadmin/staging/phpmyadmin.yaml
kubectl apply -f k8s/phpmyadmin/staging/phpmyadmin-ingress.yaml
```

<br /><br />

---

<br /><br />

# 📡 Accès MariaDB depuis le cluster

Host interne Kubernetes :

```
mariadb.staging-db.svc.cluster.local
```

Port :

```
3306
```

<br /><br />

---

<br /><br />

# 🔐 Variables d’environnement (backend)

```
DB_HOST=mariadb.staging-db.svc.cluster.local
DB_PORT=3306
DB_USER=REPLACE_ME
DB_PASSWORD=REPLACE_ME
DB_DATABASE=aetherroyale_staging
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

<br /><br />

## 🔒 Protection par BasicAuth

Créer le fichier auth :

```bash
sudo apt-get update && sudo apt-get install -y apache2-utils
htpasswd -nbB admin 'MONPASSWORD' > auth
```

Créer le secret Kubernetes :

```bash
kubectl -n staging-db create secret generic phpmyadmin-basic-auth --from-file=auth
```

<br /><br />

---

<br /><br />

## Accès

phpMyAdmin est accessible via l’Ingress sécurisé :

```
https://staging.phpmyadmin.aetherroyale.crzgames.com
```

Un login/mot de passe BasicAuth sera demandé avant l’accès.

<br /><br />

---

<br /><br />

# 🧱 Notes Architecture

### Staging

* MariaDB standalone
* 1 PVC de 10Go
* 1 seule pod
* Pas de haute disponibilité
* phpMyAdmin exposé via Ingress sécurisé

### Production (prévu)

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
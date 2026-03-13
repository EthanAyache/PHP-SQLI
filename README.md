# TP SQL Injection - BTS SIO Cybersécurité

Un projet éducatif pour enseigner les vulnérabilités **SQL Injection** aux étudiants de BTS SIO Cybersécurité. Cette application expose volontairement des failles SQLI pour des fins pédagogiques.

## 🎯 Objectifs

- Comprendre les vulnérabilités **SQL Injection** et leurs risques
- Apprendre à exploiter une requête SQL mal sécurisée
- Découvrir les bonnes pratiques de sécurité (prepared statements, sanitization)
- Pratiquer dans un environnement contrôlé et sécurisé

## 📋 Stack Technique

### Architecture

```
┌─────────────────────────────────────────────────────┐
│                 Docker Compose                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌───────────────┐         ┌───────────────────┐   │
│  │  PHP 8.1      │         │    MySQL 8.0      │   │
│  │  + Apache     │◄───────►│   (Port 3306)     │   │
│  │ (Port 8080)   │         │                   │   │
│  └───────────────┘         └───────────────────┘   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Technologies utilisées

| Composant          | Version | Rôle                                    |
| ------------------ | ------- | --------------------------------------- |
| **PHP**            | 8.1     | Traitement côté serveur, requêtes MySQL |
| **Apache**         | 2.4     | Serveur web HTTP/HTTPS                  |
| **MySQL**          | 8.0     | Base de données                         |
| **Docker**         | Latest  | Conteneurisation et orchestration       |
| **Docker Compose** | Latest  | Gestion multi-conteneurs                |

### Extensions PHP

- **MySQLi** : Interface PHP pour MySQL (compilée durant le démarrage)

## 🚀 Installation et Démarrage

### Prérequis

- **Docker Desktop** installé et en fonctionnement
- **Docker Compose** (inclus dans Docker Desktop)
- Accès à Internet (pour télécharger les images)

### Étapes de démarrage

#### 1️⃣ Cloner le projet

```bash
git clone https://github.com/Daniween/PHP-SQLI.git
cd PHP-SQLI
```

#### 2️⃣ Démarrer les containers

```bash
docker-compose up -d --build
```

**Explication** :

- `-d` : Lancer en mode détaché (arrière-plan)
- `--build` : Reconstruire les images si modifications

#### 3️⃣ Vérifier l'état

```bash
docker ps
```

Vous devriez voir :

```
web_app        RUNNING    0.0.0.0:8080->80/tcp
db_concession  RUNNING    0.0.0.0:3306->3306/tcp
```

#### 4️⃣ Accéder à l'application

Ouvrez votre navigateur : **http://localhost:8080**

### Arrêter le projet

```bash
docker-compose down
```

**Note** : Ajouter `-v` pour supprimer aussi les volumes (données) :

```bash
docker-compose down -v
```

## 📁 Structure du Projet

```
PHP-SQL-SQLI/
├── docker-compose.yml      # Configuration Docker
├── www/
│   ├── index.php          # Application principale (vulnérable)
│   └── style.css          # Feuille de styles
├── mysql/
│   └── init.sql           # Script d'initialisation BD
└── README.md              # Ce fichier
```

### Fichiers clés

#### `docker-compose.yml`

Configuration des services Docker :

- Service MySQL : Initialise la base de données
- Service PHP : Compile MySQLi et lance Apache

#### `www/index.php`

Application web avec vulnérabilité SQLI intentionnelle :

```php
// ⚠️ VULNÉRABLE !
$id = isset($_GET['id']) ? $_GET['id'] : 1;
$sql = "SELECT marque, modele, annee FROM voitures WHERE id = " . $id;
$result = $conn->query($sql);
```

**Problème** : La variable `$id` est directement concaténée sans échappement.

#### `mysql/init.sql`

Schéma et données initiales :

- Table `voitures` : Données sur les véhicules
- Table `utilisateurs` : Données sensibles (mots de passe)

## 🎓 Utilisation Pédagogique

### Scénarios d'exploitation

#### 1. Injection UNION SELECT

**Objectif** : Afficher les données de la table `utilisateurs`

```
http://localhost:8080/?id=1 UNION SELECT username, password, role FROM utilisateurs--
```

**Résultat** : Affiche les identifiants et mots de passe des administrateurs

#### 2. Injection boolean

**Objectif** : Tester une condition

```
http://localhost:8080/?id=1 OR 1=1--
```

**Résultat** : Affiche tous les véhicules

#### 3. Injection time-based

**Objectif** : Vérifier la vulnérabilité avec un délai

```
http://localhost:8080/?id=1 AND SLEEP(5)--
```

**Résultat** : Le serveur met 5 secondes pour répondre

### Points de discussion en classe

1. **Pourquoi c'est dangereux ?**
   - Accès non autorisé aux données
   - Modification/suppression de données
   - Contournement d'authentification

2. **Comment sécuriser ?**
   - Utiliser des **prepared statements**
   - Valider les entrées
   - Utiliser des **paramètres liés**

3. **Bonnes pratiques**
   ```php
   // ✅ SÉCURISÉ avec prepared statements
   $stmt = $conn->prepare("SELECT marque, modele, annee FROM voitures WHERE id = ?");
   $stmt->bind_param("i", $id);
   $stmt->execute();
   $result = $stmt->get_result();
   ```

## 🔧 Commandes Utiles

### Voir les logs

```bash
# Logs PHP/Apache
docker logs web_app

# Logs MySQL
docker logs db_concession

# Logs en temps réel
docker logs -f web_app
```

### Accéder à MySQL

```bash
docker exec -it db_concession mysql -u sio_user -ppassword123 concession_db
```

Puis dans MySQL :

```sql
SELECT * FROM voitures;
SELECT * FROM utilisateurs;
```

### Reconstruire après modifications

```bash
docker-compose down -v
docker-compose up -d --build
```

## 📊 Données Initiales

### Table `voitures`

| id  | marque  | modele  | année |
| --- | ------- | ------- | ----- |
| 1   | Toyota  | Yaris   | 2022  |
| 2   | Tesla   | Model 3 | 2023  |
| 3   | Renault | Clio    | 2019  |

### Table `utilisateurs`

| id  | username   | password                | role  |
| --- | ---------- | ----------------------- | ----- |
| 1   | admin      | SuperSecretPassword123! | admin |
| 2   | rh_manager | Pa$$w0rd_RH             | rh    |

⚠️ **Attention** : Ces mots de passe sont volontairement faibles pour la démo

## 🛡️ Avertissements de Sécurité

⚠️ **CETTE APPLICATION EST INTENTIONNELLEMENT VULNÉRABLE**

- ✋ **NE JAMAIS** utiliser en production
- ✋ **NE JAMAIS** sur Internet public
- ✋ À utiliser **UNIQUEMENT** dans un environnement éducatif contrôlé

## 📚 Ressources Complémentaires

### Lectures recommandées

- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [Prepared Statements PHP](https://www.php.net/manual/en/MySQLi.quickstart.prepared-statements.php)
- [CWE-89 SQL Injection](https://cwe.mitre.org/data/definitions/89.html)

### Pratique en ligne

- [DVWA (Damn Vulnerable Web Application)](http://dvwa.co.uk/)
- [HackTheBox](https://www.hackthebox.com/)
- [TryHackMe SQL Injection](https://tryhackme.com/)

## 👨‍💻 Support et Problèmes

### Le site n'est pas accessible

```bash
# Vérifier les containers
docker ps

# Vérifier les logs
docker logs web_app
docker logs db_concession

# Redémarrer
docker-compose restart
```

### Erreur de connexion MySQL

```bash
# Supprimer les volumes et recommencer
docker-compose down -v
docker-compose up -d --build
```

### Style CSS non chargé

Le fichier `style.css` doit être dans le même répertoire que `index.php`:

```
www/
├── index.php
└── style.css    ← À côté de index.php
```

## 📝 Licence

Ce projet est fourni à des fins éducatives uniquement.

---

**Créé pour l'enseignement de la Cybersécurité - BTS SIO**

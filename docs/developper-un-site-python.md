# Développer un site web en Python et le mettre en ligne

Ce guide décrit le processus complet : préparer l’ordinateur, écrire un site, le vérifier, puis le publier sur Internet. L’exemple est un petit livre d’or : une page affiche des messages, un formulaire en ajoute.

Le site d’exemple utilise **Flask**. Chaque étape reste visible dans le code. Les mêmes étapes valent pour Django ou FastAPI ; seuls les noms de commandes changent.

## 1. Le processus

| Étape | Ce que vous faites | Résultat |
| --- | --- | --- |
| 1 | Installer Python et Git | Les outils de base sont disponibles |
| 2 | Créer un dossier et un environnement virtuel | Les bibliothèques du projet restent isolées |
| 3 | Écrire les pages, le formulaire et la base | Le site fonctionne sur votre machine |
| 4 | Cacher les secrets et ajouter des tests | Le projet est prêt à être partagé |
| 5 | Envoyer le code sur GitHub | Une copie de référence existe |
| 6 | Lancer le site avec Gunicorn | Un serveur adapté au public répond |
| 7 | Publier sur une plateforme ou un serveur | Le site a une adresse Internet |
| 8 | Brancher le nom de domaine et HTTPS | Les visiteurs arrivent en `https://` |
| 9 | Mettre à jour par `git pull` puis redémarrage | Les corrections arrivent en ligne |

Gardez cet ordre. Déployer avant d’avoir une page qui s’ouvre en local rend les pannes difficiles à comprendre.

## 2. Choisir les outils

| Besoin | Outil retenu dans ce guide | Autre choix courant |
| --- | --- | --- |
| Langage | Python 3.11 ou plus récent | — |
| Site web | Flask | Django si vous voulez un espace d’administration et des comptes utilisateurs déjà fournis. FastAPI si le projet est surtout une API. |
| Pages HTML | Jinja2, inclus avec Flask | — |
| Base locale | SQLite, inclus avec Python | PostgreSQL dès que plusieurs personnes écrivent en même temps, ou dès que l’hébergeur efface le disque à chaque mise à jour |
| Serveur public | Gunicorn | uWSGI |
| Code source | Git et GitHub | GitLab, ou un autre forge |
| Mise en ligne simple | Une plateforme qui construit le site depuis Git | PythonAnywhere |
| Mise en ligne que vous administrez | Un serveur Ubuntu, Nginx, Certbot | Docker sur le même serveur |

Flask convient pour apprendre le processus. Django convient si le site doit grandir vite (comptes, formulaires, administration). Le déploiement reste le même : un processus Python derrière un serveur web, des variables d’environnement, HTTPS.

## 3. Préparer l’ordinateur

Vérifiez les versions :

```bash
python3 --version
git --version
```

Python doit afficher 3.11 ou plus. S’il manque, installez-le depuis le site python.org ou le gestionnaire de paquets de votre système.

Sur Ubuntu, le module `venv` est dans un paquet séparé. Sans lui, `python3 -m venv` s’arrête avec le message `ensurepip is not available`. Installez-le une fois :

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip
```

Installez aussi un éditeur (Cursor, ou un autre) et un compte GitHub.

## 4. Créer le projet

```bash
mkdir monsite
cd monsite
git init
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install flask gunicorn python-dotenv pytest
pip freeze > requirements.txt
```

`source .venv/bin/activate` est à refaire à chaque nouveau terminal. Le prompt affiche alors `(.venv)`. Sous Windows, la commande est `.venv\Scripts\activate`.

Créez `.gitignore` pour ne pas envoyer l’environnement, les secrets ni la base locale :

```gitignore
.venv/
__pycache__/
*.pyc
.env
*.db
```

`requirements.txt` contient la liste exacte des bibliothèques. Le serveur de production installera cette liste, pas « la dernière version du jour ».

## 5. Organiser les fichiers

```text
monsite/
  app.py
  requirements.txt
  .gitignore
  .env.example
  templates/
    base.html
    index.html
  static/
    style.css
  tests/
    test_app.py
```

- `app.py` reçoit les adresses (`/` , `/messages`) et parle à la base.
- `templates/` contient le HTML. Jinja2 remplace `{{ ... }}` par des données.
- `static/` contient le CSS et les images. Le navigateur les télécharge tel quel.
- `tests/` vérifie que les pages répondent encore après une modification.

## 6. Configuration et secrets

Certaines valeurs ne doivent jamais être écrites dans Git : la clé secrète des cookies, le mot de passe d’une base, une clé d’API.

Créez `.env.example`, qui sera versionné, avec des valeurs factices :

```bash
SECRET_KEY=changez-moi
```

Créez `.env` sur votre machine seulement (il est ignoré par Git) :

```bash
SECRET_KEY=dev-local-seulement
```

En production, générez une vraie clé :

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

Collez le résultat dans le `.env` du serveur, ou dans le panneau « Environment variables » de la plateforme. Ne le commitez pas.

## 7. Écrire le site

Le fichier `app.py` ci-dessous est la version à garder. Il fait quatre choses :

1. Il charge `.env` s’il existe.
2. Il ouvre SQLite et crée la table `messages` au premier passage.
3. Il affiche la page d’accueil.
4. Il enregistre un message envoyé par le formulaire.

Les requêtes SQL utilisent `?`. SQLite remplace ces marques par les valeurs. Concaténer le texte du visiteur dans la requête ouvrirait le site aux injections SQL.

```python
import os
import sqlite3
from pathlib import Path

from dotenv import load_dotenv
from flask import Flask, g, redirect, render_template, request, url_for

load_dotenv()

BASE_DIR = Path(__file__).resolve().parent


def create_app(test_config=None):
    app = Flask(__name__)
    app.config.from_mapping(
        SECRET_KEY=os.environ.get("SECRET_KEY", "dev-only-change-me"),
        DATABASE=os.environ.get("DATABASE_PATH", str(BASE_DIR / "messages.db")),
    )
    if test_config is not None:
        app.config.update(test_config)

    def get_db():
        if "db" not in g:
            g.db = sqlite3.connect(app.config["DATABASE"])
            g.db.row_factory = sqlite3.Row
        return g.db

    @app.teardown_appcontext
    def close_db(_exception):
        db = g.pop("db", None)
        if db is not None:
            db.close()

    def init_db():
        db = get_db()
        db.execute(
            """
            CREATE TABLE IF NOT EXISTS messages (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                author TEXT NOT NULL,
                body TEXT NOT NULL
            )
            """
        )
        db.commit()

    @app.before_request
    def ensure_db():
        init_db()

    @app.get("/")
    def index():
        rows = get_db().execute(
            "SELECT author, body FROM messages ORDER BY id DESC"
        ).fetchall()
        return render_template("index.html", messages=rows)

    @app.post("/messages")
    def add_message():
        author = request.form.get("author", "").strip()
        body = request.form.get("body", "").strip()
        if author and body:
            db = get_db()
            db.execute(
                "INSERT INTO messages (author, body) VALUES (?, ?)",
                (author, body),
            )
            db.commit()
        return redirect(url_for("index"))

    return app


app = create_app()
```

`templates/base.html` :

```html
<!DOCTYPE html>
<html lang="fr">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}Livre d'or{% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
  </head>
  <body>
    <main>
      {% block content %}{% endblock %}
    </main>
  </body>
</html>
```

`templates/index.html` :

```html
{% extends "base.html" %}

{% block title %}Messages{% endblock %}

{% block content %}
  <h1>Messages</h1>

  <form action="{{ url_for('add_message') }}" method="post">
    <label>
      Nom
      <input name="author" maxlength="80" required>
    </label>
    <label>
      Message
      <textarea name="body" maxlength="500" required></textarea>
    </label>
    <button type="submit">Publier</button>
  </form>

  {% if messages %}
    <ul>
      {% for message in messages %}
        <li>
          <strong>{{ message.author }}</strong>
          <p>{{ message.body }}</p>
        </li>
      {% endfor %}
    </ul>
  {% else %}
    <p>Aucun message pour le moment.</p>
  {% endif %}
{% endblock %}
```

Écrivez `{{ message.body }}` sans filtre `safe`. Jinja2 échappe alors le HTML. Un visiteur ne peut pas injecter de script dans la page.

`static/style.css` :

```css
body {
  font-family: Georgia, serif;
  margin: 2rem auto;
  max-width: 40rem;
  line-height: 1.5;
}

label {
  display: block;
  margin-bottom: 0.8rem;
}

input,
textarea,
button {
  font: inherit;
}

input,
textarea {
  display: block;
  width: 100%;
  margin-top: 0.3rem;
}

li {
  margin-bottom: 1rem;
}
```

## 8. Voir le site sur votre machine

Dans le dossier du projet, avec l’environnement virtuel activé :

```bash
flask --app app run --debug
```

Ouvrez http://127.0.0.1:5000 . Publiez un message. Rechargez la page : il est toujours là, dans le fichier `messages.db`.

`--debug` recharge le code quand vous sauvegardez et affiche les erreurs dans le navigateur. Réservez ce mode à votre machine. En public, une page de débogage montre le code source et les variables.

Arrêtez le serveur avec Ctrl+C.

## 9. Vérifier avec des tests

`tests/test_app.py` :

```python
import pytest

from app import create_app


@pytest.fixture
def client(tmp_path):
    app = create_app(
        {
            "TESTING": True,
            "DATABASE": str(tmp_path / "test.db"),
            "SECRET_KEY": "test",
        }
    )
    return app.test_client()


def test_page_accueil(client):
    response = client.get("/")
    assert response.status_code == 200
    text = response.get_data(as_text=True)
    assert "Messages" in text
    assert "Aucun message" in text


def test_ajout_message(client):
    response = client.post(
        "/messages",
        data={"author": "Awa", "body": "Bonjour"},
        follow_redirects=True,
    )
    assert response.status_code == 200
    assert "Bonjour" in response.get_data(as_text=True)
```

Lancez :

```bash
python -m pytest
```

Les deux tests doivent passer. `python -m pytest` lance les tests avec le dossier du projet sur le chemin Python, ce qui permet `from app import create_app`. Ajoutez un test à chaque nouveau comportement important (une page, une règle de formulaire, un calcul). Un test qui échoue avant la mise en ligne vaut mieux qu’une page cassée chez les visiteurs.

## 10. Enregistrer le code avec Git

```bash
git add app.py requirements.txt .gitignore .env.example templates static tests
git commit -m "Ajouter le livre d'or"
```

Créez un dépôt vide sur GitHub, puis :

```bash
git branch -M main
git remote add origin https://github.com/VOTRE_COMPTE/monsite.git
git push -u origin main
```

Vérifiez sur GitHub que `.env` et `messages.db` sont absents. S’ils apparaissent, retirez-les de Git, ajoutez-les au `.gitignore`, changez la clé secrète, puis poussez à nouveau. Une clé déjà poussée doit être considérée comme publique.

## 11. Préparer la version publique

`flask run` est un serveur de développement. Il sert un seul visiteur à la fois et n’est pas fait pour rester allumé. En public, Gunicorn lance l’objet `app` défini à la fin de `app.py` :

```bash
gunicorn --workers 1 --bind 127.0.0.1:8000 app:app
```

Ouvrez http://127.0.0.1:8000 pour confirmer.

`--workers 1` est volontaire. SQLite supporte mal plusieurs processus qui écrivent en même temps. Quand vous passerez à PostgreSQL, montez à 2 ou 3 workers.

Avant de publier, contrôlez aussi :

- `SECRET_KEY` vient de l’environnement, avec une longue valeur aléatoire.
- Le mode debug est éteint.
- `python -m pytest` passe.
- La base de production a une sauvegarde. Copiez `messages.db`, ou utilisez les sauvegardes de PostgreSQL.

SQLite dans un fichier convient sur un serveur dont le disque vous appartient. Sur beaucoup de plateformes, le disque est recréé à chaque déploiement : les messages disparaîtraient. Dans ce cas, créez une base PostgreSQL chez l’hébergeur et adaptez la connexion. Le reste du processus (Git, Gunicorn, HTTPS, variables d’environnement) ne change pas.

## 12. Mettre le site en ligne sur une plateforme

C’est le chemin le plus court. La plateforme clone votre Git, installe `requirements.txt`, puis lance Gunicorn. Render est un exemple ; Railway, Fly.io et les offres « application web Python » des autres hébergeurs suivent les mêmes réglages.

1. Poussez la branche `main` sur GitHub.
2. Créez un **Web Service** relié à ce dépôt.
3. Langage : Python.
4. Commande de construction : `pip install -r requirements.txt`
5. Commande de démarrage : `gunicorn --workers 1 --bind 0.0.0.0:$PORT app:app`
6. Ajoutez la variable `SECRET_KEY` dans le panneau des variables d’environnement. La plateforme fournit en général `PORT`.
7. Lancez le déploiement et ouvrez l’adresse `*.onrender.com` (ou l’équivalent) quand le journal indique que Gunicorn écoute.

Si la page ne s’ouvre pas, lisez le journal de construction puis le journal d’exécution. Les causes fréquentes : `requirements.txt` incomplet, `app:app` qui ne correspond pas au nom du fichier, ou `PORT` ignoré (le service doit écouter sur `0.0.0.0` et le port reçu).

### PythonAnywhere

PythonAnywhere est une autre option simple, souvent utilisée pour apprendre.

1. Envoyez le projet (Git ou l’outil de fichiers) dans `/home/VOTRE_COMPTE/monsite`.
2. Créez un environnement virtuel dans ce dossier et installez `requirements.txt`.
3. Onglet Web, créez une application Flask.
4. Dans le fichier WSGI, pointez vers votre application :

```python
import sys

path = "/home/VOTRE_COMPTE/monsite"
if path not in sys.path:
    sys.path.append(path)

from app import app as application
```

5. Déclarez les fichiers statiques : URL `/static/` vers `/home/VOTRE_COMPTE/monsite/static`.
6. Ajoutez `SECRET_KEY` dans le fichier WSGI ou dans les variables du compte, avant l’import de `app` si la clé est lue à l’import. Avec le code de ce guide, `load_dotenv()` lit un fichier `.env` placé dans le dossier du projet.
7. Cliquez sur Reload et ouvrez l’adresse fournie.

## 13. Mettre le site en ligne sur votre serveur

Choisissez cette voie si vous voulez le nom de domaine, le système et les sauvegardes sous votre contrôle. Les commandes supposent Ubuntu, un utilisateur `ubuntu` qui peut utiliser `sudo`, et le dépôt déjà poussé sur GitHub.

### Créer le serveur et ouvrir les ports

Chez l’hébergeur (Hetzner, OVH, DigitalOcean, ou un autre), créez une machine Ubuntu 24.04 et notez son adresse IPv4. Connectez-vous :

```bash
ssh ubuntu@ADRESSE_IP
```

Autorisez SSH et le web, puis activez le pare-feu :

```bash
sudo ufw allow OpenSSH
sudo ufw allow "Nginx Full"
sudo ufw enable
```

### Installer les paquets et le code

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip nginx certbot python3-certbot-nginx
sudo mkdir -p /var/www
sudo chown ubuntu:ubuntu /var/www
cd /var/www
git clone https://github.com/VOTRE_COMPTE/monsite.git
cd monsite
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Créez `/var/www/monsite/.env` sur le serveur (pas dans Git) :

```bash
SECRET_KEY=la-cle-generee-avec-secrets.token_hex
```

### Lancer Gunicorn au démarrage

Créez `/etc/systemd/system/monsite.service` :

```ini
[Unit]
Description=Livre d'or Gunicorn
After=network.target

[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/var/www/monsite
EnvironmentFile=/var/www/monsite/.env
ExecStart=/var/www/monsite/.venv/bin/gunicorn --workers 1 --bind 127.0.0.1:8000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now monsite
sudo systemctl status monsite
```

`active (running)` confirme que Python répond en local sur le port 8000. Le port 8000 n’est pas ouvert sur Internet : seul Nginx, sur la même machine, lui parle.

### Placer Nginx devant

Créez `/etc/nginx/sites-available/monsite` :

```nginx
server {
    listen 80;
    server_name exemple.fr www.exemple.fr;

    location /static/ {
        alias /var/www/monsite/static/;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Remplacez `exemple.fr` par votre domaine.

```bash
sudo ln -s /etc/nginx/sites-available/monsite /etc/nginx/sites-enabled/monsite
sudo nginx -t
sudo systemctl reload nginx
```

`nginx -t` doit dire que la syntaxe est correcte avant le rechargement.

## 14. Nom de domaine et HTTPS

Chez le registrar du domaine, créez un enregistrement DNS :

| Type | Nom | Valeur |
| --- | --- | --- |
| A | `@` | Adresse IPv4 du serveur |
| A | `www` | La même adresse |

Attendez que le nom résolve vers le serveur (`dig exemple.fr` ou un test depuis le navigateur). Ensuite, sur le serveur :

```bash
sudo certbot --nginx -d exemple.fr -d www.exemple.fr
```

Certbot obtient un certificat Let’s Encrypt et modifie Nginx pour rediriger le HTTP vers HTTPS. Le renouvellement est installé automatiquement. Vérifiez-le avec :

```bash
sudo certbot renew --dry-run
```

Ouvrez `https://exemple.fr`. Le cadenas du navigateur doit être présent, et un message publié doit réapparaître après un rechargement.

Sur une plateforme, le HTTPS de l’adresse fournie (`https://….onrender.com`) est déjà actif. Pour votre propre domaine, ajoutez-le dans le panneau de la plateforme et créez l’enregistrement DNS qu’elle indique (souvent un `CNAME`).

## 15. Publier une mise à jour

Sur votre machine :

```bash
git add -u
git commit -m "Décrire le changement"
git push
```

Sur une plateforme, le push déclenche en général un nouveau déploiement. Sur votre serveur :

```bash
cd /var/www/monsite
git pull
source .venv/bin/activate
pip install -r requirements.txt
sudo systemctl restart monsite
```

Rejouez `python -m pytest` avant le push. Après le redémarrage, ouvrez la page et le journal :

```bash
sudo journalctl -u monsite -n 50 --no-pager
```

Si la nouvelle version est cassée, revenez au commit précédent avec `git checkout` du commit connu, puis redémarrez le service. Pour une base, restaurez la copie de sauvegarde faite avant la mise à jour.

Sauvegardez `messages.db` régulièrement, par exemple :

```bash
sqlite3 /var/www/monsite/messages.db ".backup /var/backups/messages-$(date +%F).db"
```

## 16. Liste de contrôle

Avant d’annoncer l’adresse :

- La page d’accueil s’ouvre en local, et le formulaire ajoute un message.
- `python -m pytest` passe.
- `.env`, la base et `.venv` ne sont pas dans Git.
- `SECRET_KEY` de production est longue et différente de celle de développement.
- Gunicorn écoute, le mode debug est éteint.
- HTTPS répond, et HTTP redirige vers HTTPS.
- Une sauvegarde de la base existe, et vous savez comment la restaurer.
- Vous avez ouvert le site dans un navigateur, publié un message, rechargé la page.

## 17. Faire grandir le projet

Le processus ne change pas quand le site grossit. Vous ajoutez des pages dans `templates/`, des routes dans `app.py`, des tests, puis vous déployez de la même façon.

Passez à l’étape suivante seulement quand le besoin apparaît :

- **Comptes et administration** : Django, ou Flask avec des extensions dédiées.
- **Plusieurs rédacteurs en même temps** : PostgreSQL à la place de SQLite.
- **Formulaires publics** : jeton CSRF (Flask-WTF) pour qu’un autre site ne puisse pas soumettre le formulaire à la place du visiteur.
- **Mises à jour automatiques** : un workflow GitHub Actions qui lance `python -m pytest` à chaque push, puis déploie seulement si les tests passent.
- **Plusieurs services** : Docker, pour décrire dans un fichier l’application, la base et Nginx. Le flux reste : construire l’image, la lancer, exposer le port 80 et 443.

Chaque ajout suit la même boucle : coder, tester en local, commit, déployer, vérifier la page en ligne.

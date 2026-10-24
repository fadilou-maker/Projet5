# Utiliser GitHub, GitHub Desktop et GitLab

Ce guide couvre le travail quotidien sur un projet : créer le dépôt, enregistrer un changement, l’envoyer, mettre à jour votre copie, proposer une modification, et suivre les tâches. Les mêmes gestes existent sur GitHub et sur GitLab. GitHub Desktop est l’application de bureau pour GitHub.

Les boutons cités sont ceux de la documentation officielle, en anglais. Si votre interface est en français, le libellé change, le geste reste le même.

## 1. Quatre noms, quatre rôles

| Nom | Rôle | Adresse |
| --- | --- | --- |
| Git | Programme installé sur l’ordinateur. Il enregistre l’historique des fichiers. | [Installer Git](https://docs.github.com/en/get-started/git-basics/set-up-git) |
| GitHub | Site qui héberge les dépôts Git, les discussions et les demandes de fusion. | [github.com](https://github.com/) |
| GitHub Desktop | Application qui fait les commandes Git à la souris, reliée à un compte GitHub. | [desktop.github.com](https://desktop.github.com/) |
| GitLab | Autre site d’hébergement Git. Un « projet » GitLab est un dépôt. Une entreprise peut aussi installer GitLab sur ses propres serveurs. | [gitlab.com](https://gitlab.com/) |

Git reste le même outil dans les trois cas. Changer de site ne change pas `git add`, `git commit`, `git push` ni `git pull`. Seuls l’adresse du dépôt, le nom de la demande de fusion et quelques boutons changent.

| Idée | GitHub | GitLab |
| --- | --- | --- |
| Le dossier du code en ligne | Repository, dépôt | Project, projet |
| La demande pour fusionner une branche | Pull request | Merge request |
| La liste de tâches | Issues, et le tableau [GitHub Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects) | [Issues](https://docs.gitlab.com/user/project/issues/) et [issue boards](https://docs.gitlab.com/user/project/issue_board/) |
| Plusieurs dépôts regroupés | Organisation | [Groupe](https://docs.gitlab.com/user/group/) |

GitHub Desktop crée des pull requests GitHub. Il ne crée pas une merge request GitLab. Vous pouvez quand même l’utiliser pour cloner une adresse GitLab, commit, push et pull. La fusion se valide ensuite sur le site GitLab.

## 2. Le cycle à répéter

Chaque modification suit le même ordre.

1. Mettre à jour votre copie avec ce qui est déjà en ligne.
2. Créer une branche si le changement n’est pas minuscule, ou si d’autres personnes travaillent sur le projet.
3. Modifier les fichiers.
4. Choisir les fichiers à enregistrer.
5. Créer un commit, avec une phrase qui dit ce qui change.
6. Pousser la branche vers GitHub ou GitLab.
7. Ouvrir une pull request ou une merge request.
8. Après la fusion, remettre votre branche principale à jour.

Un commit existe d’abord seulement sur votre ordinateur. Le push le rend visible aux autres. Tant que vous n’avez pas poussé, personne d’autre ne voit le commit.

## 3. Créer les comptes

- GitHub : [github.com/signup](https://github.com/signup)
- GitLab : [gitlab.com/users/sign_up](https://gitlab.com/users/sign_up)

Utilisez une adresse e-mail que vous consultez. Le nom d’utilisateur apparaît dans les adresses de vos dépôts, par exemple `https://github.com/VOTRE_COMPTE/mon-projet`.

Vérifiez l’e-mail reçu. Sans cette vérification, certains envois sont bloqués.

## 4. Installer les outils

1. Installez Git : [Set up Git](https://docs.github.com/en/get-started/git-basics/set-up-git). Sur Ubuntu : `sudo apt install git`.
2. Installez GitHub Desktop depuis [desktop.github.com](https://desktop.github.com/), puis suivez [Getting started with GitHub Desktop](https://docs.github.com/en/desktop/overview/getting-started-with-github-desktop).
3. Ouvrez GitHub Desktop et connectez-vous au compte GitHub. L’application mémorise l’accès pour les dépôts GitHub en HTTPS.

Vérifiez dans un terminal :

```bash
git --version
```

Dites à Git quel nom et quelle adresse mettre sur vos commits. L’adresse doit être celle du compte GitHub ou GitLab, pour que le site relie le commit à vous.

```bash
git config --global user.name "Votre Nom"
git config --global user.email "vous@exemple.com"
```

## 5. Se connecter sans retaper un mot de passe

Deux méthodes existent.

**SSH, recommandée par GitLab.** Vous créez une clé sur l’ordinateur et vous déposez la clé publique sur le site. Les adresses ressemblent à `git@github.com:VOTRE_COMPTE/mon-projet.git` ou `git@gitlab.com:VOTRE_COMPTE/mon-projet.git`.

**HTTPS.** L’adresse commence par `https://`. GitHub Desktop gère cet accès pour GitHub après la connexion. En ligne de commande, le mot de passe du compte ne convient pas : il faut un jeton d’accès.

- GitHub : [personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- GitLab : [personal access tokens](https://docs.gitlab.com/user/profile/personal_access_tokens/)

### Créer une clé SSH

Les guides officiels détaillent chaque système :

- GitHub : [générer une clé](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) puis [l’ajouter au compte](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
- GitLab : [Use SSH keys with GitLab](https://docs.gitlab.com/user/ssh/)

Le geste commun :

```bash
ssh-keygen -t ed25519 -C "vous@exemple.com"
```

Appuyez sur Entrée pour accepter le fichier proposé. Une phrase secrète est utile : elle protège la clé si quelqu’un copie votre ordinateur.

Affichez la clé **publique** seulement, celle qui finit par `.pub`, et copiez-la :

```bash
cat ~/.ssh/id_ed25519.pub
```

Collez cette ligne dans GitHub, sous Settings, SSH and GPG keys, New SSH key, type Authentication. Sur GitLab : préférences, SSH Keys, Add new key. Ne copiez jamais le fichier sans `.pub`. C’est la clé privée.

Vérifiez :

```bash
ssh -T git@github.com
ssh -T git@gitlab.com
```

GitHub répond avec votre nom d’utilisateur. GitLab répond `Welcome to GitLab`. Une demande de confirmation de l’empreinte au premier essai est normale : répondez `yes`.

Une seule clé peut servir aux deux sites. Ajoutez la même clé publique sur chaque compte.

## 6. Les commandes qui marchent sur GitHub et sur GitLab

Placez-vous dans le dossier du projet avant ces commandes. Remplacez l’adresse par celle du bouton **Code** du dépôt.

### Récupérer un projet qui existe déjà

```bash
git clone git@github.com:VOTRE_COMPTE/mon-projet.git
cd mon-projet
```

Pour GitLab, l’adresse est du type `git@gitlab.com:VOTRE_COMPTE/mon-projet.git`. Le clone télécharge les fichiers et tout l’historique, puis mémorise le lien sous le nom `origin`.

Documentation : [cloner sur GitHub](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository), [cloner sur GitLab](https://docs.gitlab.com/topics/git/clone/).

### Envoyer un projet déjà présent sur l’ordinateur

Créez d’abord un dépôt vide sur le site, sans README si votre dossier en a déjà un. Ensuite :

```bash
cd mon-projet
git init
git branch -M main
git add .
git status
git commit -m "Premier enregistrement du projet"
git remote add origin git@github.com:VOTRE_COMPTE/mon-projet.git
git push -u origin main
```

`git add .` prépare tous les fichiers du dossier. Regardez `git status` avant le commit : un `.env`, un dossier `.venv` ou `node_modules` ne doit pas apparaître. `-u` mémorise le lien entre votre branche `main` et `origin/main`. Les prochains envois sont alors un simple `git push`.

Sur GitLab, la même suite fonctionne avec l’adresse GitLab. Le site peut aussi créer le projet au premier push : [Create a project with git push](https://docs.gitlab.com/topics/git/project/). Créer le projet dans l’interface reste le chemin le plus clair : [Create a project](https://docs.gitlab.com/user/project/).

### Enregistrer un changement

```bash
git status
git add chemin/du/fichier
git commit -m "Décrire le changement en une phrase"
git push
```

- `git status` montre ce qui a changé, ce qui est prêt, et la branche en cours.
- `git add` choisit ce qui entre dans le prochain commit. Vous pouvez le lancer plusieurs fois.
- `git commit` enregistre un instantané sur votre ordinateur.
- `git push` envoie les commits qui ne sont pas encore sur le site.

Pour préparer tout le dossier : `git add .`

Le message de commit dit pourquoi le changement existe. « Correction du formulaire de contact » est utile. « update » ne l’est pas.

Détail GitLab : [Stage, commit, and push changes](https://docs.gitlab.com/topics/git/commit/). Premier parcours guidé : [Make your first Git commit](https://docs.gitlab.com/tutorials/make_first_git_commit/).

### Mettre votre copie à jour

```bash
git pull
```

Cette commande télécharge les commits de la branche suivie et les fusionne dans vos fichiers. Faites-le avant de commencer à travailler, et avant un push si quelqu’un d’autre a poussé entre-temps.

`git fetch` télécharge sans modifier vos fichiers. `git pull` fait un fetch, puis fusionne. Pour voir l’historique :

```bash
git log --oneline -10
```

### Créer une branche et la publier

```bash
git switch main
git pull
git switch -c fonction-contact
```

Travaillez, committez, puis :

```bash
git push -u origin fonction-contact
```

Revenir sur la branche principale :

```bash
git switch main
```

`git switch -c` crée la branche et vous y place. Les anciens guides écrivent `git checkout -b`, qui fait la même chose.

## 7. GitHub dans le navigateur

Guide court officiel : [Quickstart for repositories](https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories).

### Créer un dépôt

1. En haut à droite, menu **+**, puis **New repository**.
2. Donnez un nom court, sans espaces. `mon-projet` convient.
3. Choisissez Public ou Private. Un dépôt privé n’est visible que par vous et les personnes invitées.
4. Cochez **Add a README** si le projet commence sur GitHub. Décochez-le si vous allez envoyer un dossier qui a déjà des fichiers, pour éviter deux historiques différents.
5. **Create repository**.

### Modifier un fichier sans installer Git

Sur la page du fichier : crayon **Edit**. Changez le texte, descendez jusqu’au message de commit, puis **Commit changes**. Utile pour une faute dans le README. Pour du vrai développement, travaillez sur votre ordinateur.

### Cloner

Bouton vert **Code**, onglet SSH ou HTTPS, copiez l’adresse, puis `git clone` dans le terminal. Ou passez par GitHub Desktop, section 8.

### Issues

Une issue décrit un bug, une idée ou une tâche. Onglet **Issues**, **New issue**. Titre clair, contexte, ce que vous attendez. [Creating an issue](https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/creating-an-issue).

### GitHub Projects

Un Project est un tableau. Il range des issues et des pull requests en colonnes, par exemple À faire, En cours, Terminé. Ce n’est pas une copie du code.

1. Sur votre profil ou dans l’organisation, **Projects**, **New project**.
2. Ouvrez une issue ou une pull request, panneau **Projects**, et ajoutez-la au tableau.
3. Déplacez la carte quand le travail avance.

[About Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects).

### Releases

Une release marque une version publiée, par exemple `v1.0.0`, à partir d’un tag Git. [About releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases).

### Inviter quelqu’un

**Settings**, **Collaborators**, **Add people**. La personne reçoit un e-mail. Après acceptation, elle peut cloner et pousser selon le rôle choisi. [Gérer les accès](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/managing-teams-and-people-with-access-to-your-repository).

### Fork

Si vous n’avez pas le droit d’écrire dans un dépôt, **Fork** en crée une copie sur votre compte. Vous poussez sur votre copie, puis vous ouvrez une pull request vers le dépôt d’origine. [Fork a repo](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo).

## 8. GitHub Desktop

Documentation générale : [GitHub Desktop](https://docs.github.com/en/desktop).

L’écran a trois zones utiles :

- **Current Branch**, en haut : la branche en cours, le bouton pour en créer une, et Fetch, Pull ou Push.
- La liste de gauche : les fichiers modifiés. L’onglet **History** montre les commits déjà faits.
- En bas à gauche : **Summary** et **Description**, puis **Commit to nom-de-branche**.

### Créer un dépôt

[Creating your first repository using GitHub Desktop](https://docs.github.com/en/desktop/overview/creating-your-first-repository-using-github-desktop).

1. **File**, **New repository**.
2. Name : le nom du dossier et du futur dépôt.
3. Local path : l’endroit sur le disque. GitHub Desktop crée le dossier à l’intérieur de ce chemin.
4. Cochez **Initialize this repository with a README**.
5. Git ignore : choisissez le langage si vous le connaissez, par exemple Python. Cela crée un `.gitignore`.
6. **Create repository**.
7. Le bouton du haut devient **Publish repository**. Cliquez-le, choisissez le compte, Public ou Private, puis publiez.

Le projet est alors sur GitHub et sur votre ordinateur.

Pour un dossier qui existe déjà : **File**, **Add local repository**. S’il n’y a pas encore de Git dedans, Desktop propose de le créer. Ensuite, **Publish repository**. [Ajouter un projet existant](https://docs.github.com/en/desktop/adding-and-cloning-repositories/adding-an-existing-project-to-github-using-github-desktop).

### Cloner

[Cloning and forking repositories from GitHub Desktop](https://docs.github.com/en/desktop/adding-and-cloning-repositories/cloning-and-forking-repositories-from-github-desktop).

1. **File**, **Clone repository**.
2. Onglet **GitHub.com** pour vos dépôts, ou **URL** pour coller une adresse, y compris une adresse GitLab.
3. Choisissez le dossier local.
4. **Clone**.

### Commit

[Committing and reviewing changes](https://docs.github.com/en/desktop/contributing-and-collaborating-using-github-desktop/making-changes-in-a-branch/committing-and-reviewing-changes-to-your-project).

1. Modifiez les fichiers dans votre éditeur et enregistrez.
2. Revenez à GitHub Desktop. Les fichiers apparaissent à gauche. Cliquez un fichier pour voir le diff, ancien texte en rouge, nouveau en vert.
3. Décochez un fichier s’il ne doit pas entrer dans ce commit.
4. Summary : une phrase courte. Description : le détail, si besoin.
5. **Commit to nom-de-branche**.

Le commit est local. Le site GitHub n’a pas encore changé.

### Push

[Pushing changes to GitHub from GitHub Desktop](https://docs.github.com/en/desktop/contributing-and-collaborating-using-github-desktop/making-changes-in-a-branch/pushing-changes-to-github-from-github-desktop).

1. En haut, **Push origin**.
2. Si le site a des commits que vous n’avez pas, Desktop ouvre **New Commits on Remote**. Cliquez **Fetch**, mettez votre branche à jour, puis poussez.

La première publication d’une nouvelle branche affiche **Publish branch** plutôt que Push origin.

GitHub bloque un fichier de plus de 100 MiB. GitHub Desktop refuse aussi un push de plus de 2 GiB. Les fichiers plus lourds passent par [Git Large File Storage](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-git-large-file-storage). Avec GitHub Desktop : [About Git Large File Storage and GitHub Desktop](https://docs.github.com/en/desktop/configuring-and-customizing-github-desktop/about-git-large-file-storage-and-github-desktop).

### Mettre à jour

[Syncing your branch in GitHub Desktop](https://docs.github.com/en/desktop/working-with-your-remote-repository-on-github-or-github-enterprise/syncing-your-branch-in-github-desktop).

1. **Current Branch**, choisissez la branche à mettre à jour.
2. **Fetch origin** pour regarder s’il y a du nouveau.
3. **Pull origin** pour l’intégrer. **Pull origin with rebase** est une variante : elle réécrit vos commits locaux par-dessus ceux du site. Restez sur **Pull origin** tant que vous ne savez pas pourquoi vous voulez un rebase.

Pull ne met à jour que votre ordinateur. Pour que les autres voient votre travail, il faut ensuite un push.

### Branches

[Managing branches in GitHub Desktop](https://docs.github.com/en/desktop/making-changes-in-a-branch/managing-branches-in-github-desktop).

1. **Current Branch**, sélectionnez la branche de départ, en général `main`.
2. **New Branch**.
3. Donnez un nom, par exemple `fonction-contact`.
4. **Create Branch**.
5. Après le premier commit, **Publish branch**.

Pour changer de branche : **Current Branch**, puis le nom. S’il reste des modifications non commitées, Desktop demande de les laisser sur la branche actuelle ou de les emporter.

Pour ramener `main` dans votre branche : **Current Branch**, **Choose a branch to merge into**, choisissez `main`, **Merge**. Puis **Push origin**.

### Pull request

[Creating an issue or pull request from GitHub Desktop](https://docs.github.com/en/desktop/working-with-your-remote-repository-on-github-or-github-enterprise/creating-an-issue-or-pull-request-from-github-desktop).

1. La branche est publiée et poussée.
2. **Preview Pull Request**.
3. Vérifiez la base, en général `main`.
4. **Create Pull Request**. Le navigateur s’ouvre.
5. Titre, description, puis **Create pull request**. Le menu permet aussi **Create draft pull request** si le travail n’est pas fini.

Les commits poussés ensuite sur la même branche s’ajoutent à cette pull request.

Pour une issue : menu **Repository**, **Create Issue on GitHub**.

### Historique et annulation locale

Onglet **History** : chaque commit, ses fichiers, son auteur. Clic droit sur un commit pour des actions comme créer une branche à partir de ce point.

Pour retirer un fichier du prochain commit sans effacer votre travail : dans **Changes**, clic droit, **Discard changes** seulement si vous voulez vraiment perdre la modification. Cette action ne se récupère pas facilement.

## 9. GitLab

GitLab.com est le service public. Une adresse du type `gitlab.entreprise.com` est un GitLab installé par une organisation : les boutons sont les mêmes, le domaine change. Remplacez alors `gitlab.com` par ce domaine dans les adresses SSH.

Vue d’ensemble de Git : [Get started with Git](https://docs.gitlab.com/topics/git/get_started/).

### Créer un projet

[Create a project](https://docs.gitlab.com/user/project/).

1. Bouton **Create new**, **New project/repository**.
2. **Create blank project**.
3. Project name : `Mon projet`. Le slug, par exemple `mon-projet`, devient l’adresse.
4. Choisissez le groupe, ou votre nom si le projet est personnel.
5. Visibility : Private, Internal ou Public.
6. Cochez **Initialize repository with a README** pour un projet neuf.
7. **Create project**.

### Cloner, commit, push

1. Sur la page du projet, **Code**, **Clone with SSH**.
2. Dans le terminal :

```bash
git clone git@gitlab.com:VOTRE_COMPTE/mon-projet.git
cd mon-projet
git switch -c exemple-texte
```

3. Modifiez `README.md`, puis :

```bash
git add README.md
git commit -m "Ajouter une phrase au README"
git push -u origin exemple-texte
```

Le parcours complet est le tutoriel [Make your first Git commit](https://docs.gitlab.com/tutorials/make_first_git_commit/).

Vous pouvez aussi éditer sur le site avec le [Web Editor](https://docs.gitlab.com/user/project/repository/web_editor/) : ouvrez le fichier, modifiez, et commitez. Pratique pour une petite correction. Le développement habituel reste sur votre ordinateur.

### Merge request

C’est l’équivalent GitLab de la pull request. [Create merge requests](https://docs.gitlab.com/user/project/merge_requests/creating_merge_requests/).

Après le push, GitLab affiche souvent une bannière **Create merge request**. Sinon : menu **Merge requests**, **New merge request**, choisissez la branche source et la branche cible `main`, **Compare branches and continue**.

Remplissez le titre et la description. **Create merge request**. Quand c’est accepté, **Merge**.

En ligne de commande, au moment du push :

```bash
git push -o merge_request.create -u origin exemple-texte
```

Les options sont décrites dans [Stage, commit, and push changes](https://docs.gitlab.com/topics/git/commit/).

Une merge request se met à jour toute seule quand vous poussez de nouveaux commits sur la même branche.

### Mettre à jour

```bash
git switch main
git pull origin main
```

Puis, pour intégrer `main` dans votre branche de travail :

```bash
git switch exemple-texte
git merge main
git push
```

### Issues, tableaux et groupes

- **Issues** : **New issue**, titre et description. [Issues](https://docs.gitlab.com/user/project/issues/).
- **Issue boards** : colonnes Ouvert, En cours, Fermé, ou les vôtres. [Issue boards](https://docs.gitlab.com/user/project/issue_board/).
- **Groupes** : un groupe contient plusieurs projets, par exemple un groupe de classe avec un projet par personne. [Groups](https://docs.gitlab.com/user/group/).
- **Releases** : une version nommée du projet. [Releases](https://docs.gitlab.com/user/project/releases/).
- **Branches** : liste, protection, suppression. [Branches](https://docs.gitlab.com/user/project/repository/branches/).

Pour inviter quelqu’un : **Manage**, **Members**, **Invite members**. Choisissez un rôle. Developer peut pousser des branches. Maintainer peut fusionner et modifier davantage de réglages.

### GitLab avec GitHub Desktop

1. Copiez l’adresse SSH du projet GitLab.
2. Dans GitHub Desktop : **File**, **Clone repository**, onglet **URL**, collez l’adresse.
3. Commit, **Push origin** et **Pull origin** fonctionnent si la clé SSH est enregistrée sur GitLab.
4. Créez la merge request sur gitlab.com. Le bouton **Preview Pull Request** sert aux dépôts GitHub.

## 10. Proposer et fusionner un changement

Même histoire des deux côtés. Guide GitHub : [Pull request quickstart](https://docs.github.com/en/pull-requests/get-started/pull-request-quickstart). Guide GitLab : [Merge requests](https://docs.gitlab.com/user/project/merge_requests/).

1. Partez d’une branche à jour.
2. Un sujet par branche. Évitez de mélanger une correction et une nouvelle page.
3. Poussez la branche.
4. Ouvrez la pull request ou la merge request vers `main`.
5. Expliquez ce qui change et comment le vérifier.
6. Lisez les commentaires, corrigez, et poussez sur la même branche.
7. Fusionnez quand c’est accepté.
8. Sur votre ordinateur : `git switch main` puis `git pull`.
9. Supprimez la branche de travail si le travail est terminé. Sur GitHub Desktop : **Current Branch**, clic droit, delete. En commande, après être revenu sur `main` : `git branch -d fonction-contact` puis `git push origin --delete fonction-contact`.

## 11. Mettre un projet à jour : quatre situations

### Quelqu’un a poussé sur la branche que vous utilisez

Le push est refusé, souvent avec `rejected` et `fetch first`.

```bash
git pull
git push
```

Dans GitHub Desktop : **Fetch origin**, **Pull origin**, réglez un conflit s’il apparaît, **Push origin**.

### Vous travaillez sur deux ordinateurs

Avant de partir : commit et push. Sur l’autre machine : pull. Ne copiez pas le dossier à la main d’un ordinateur à l’autre : les historiques divergent.

### `main` a avancé et votre branche est en retard

```bash
git switch main
git pull
git switch fonction-contact
git merge main
```

Réglez les conflits, committez si Git vous le demande, puis `git push`.

Dans GitHub Desktop : placez-vous sur votre branche, **Choose a branch to merge into**, choisissez `main`, **Merge**, puis **Push origin**.

### Vous avez un fork et l’original a changé

Sur GitHub, le bouton **Sync fork** de la page du fork ramène les commits du dépôt d’origine. En commande, une fois le lien ajouté :

```bash
git remote add upstream git@github.com:COMPTE_ORIGINAL/mon-projet.git
git fetch upstream
git switch main
git merge upstream/main
git push origin main
```

`upstream` est le dépôt d’origine. `origin` reste votre fork. [Syncing a fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/syncing-a-fork).

Sur GitLab, ajoutez de la même façon le projet d’origine comme remote `upstream`, puis fetch, merge et push vers votre copie.

## 12. Conflits

Un conflit arrive quand deux commits modifient les mêmes lignes. Git s’arrête et marque le fichier :

```text
<<<<<<< HEAD
votre version
=======
l'autre version
>>>>>>> nom-de-branche
```

Ouvrez le fichier, gardez le bon texte, effacez les lignes `<<<<<<<`, `=======` et `>>>>>>>`, enregistrez. Puis :

```bash
git add chemin/du/fichier
git commit
git push
```

GitHub Desktop signale le conflit et vous laisse l’ouvrir dans l’éditeur. Après correction, le fichier réapparaît dans les changements : committez, puis poussez.

Méthode détaillée : [résoudre un conflit en ligne de commande](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line).

## 13. Fichiers à ne pas envoyer

Créez un `.gitignore` à la racine avant le premier commit. GitHub peut en proposer un à la création du dépôt. GitHub Desktop a la liste **Git ignore** à la création du dépôt. Explication : [Ignoring files](https://docs.github.com/en/get-started/git-basics/ignoring-files).

Exemple pour un projet Python :

```gitignore
.venv/
__pycache__/
.env
*.db
```

Pour un projet JavaScript, ignorez aussi `node_modules/`.

Si un secret a déjà été poussé, le retirer du dernier fichier ne suffit pas : il reste dans l’historique. Changez le mot de passe ou la clé, puis nettoyez l’historique avec l’aide de la documentation du site. Considérez le secret comme public dès qu’il a été poussé sur un dépôt, même privé, si d’autres personnes y ont accès.

## 14. Revenir en arrière

| Situation | Geste |
| --- | --- |
| Le fichier a changé, rien n’est encore dans `git add` | `git restore chemin/du/fichier` |
| Le fichier est prêt pour le commit, vous voulez le retirer de cette liste sans perdre le texte | `git restore --staged chemin/du/fichier` |
| Le dernier commit n’a pas été poussé et vous voulez le refaire | `git reset --soft HEAD~1`, puis un nouveau commit |
| Le commit est déjà poussé et vous voulez annuler son effet | Un nouveau commit qui remet l’ancien contenu. L’historique garde les deux. |

Évitez `git push --force` sur `main`. Cette commande réécrit ce que les autres ont déjà récupéré. Elle sert dans des cas précis, sur une branche à vous, quand toute l’équipe est d’accord.

Dans GitHub Desktop, annuler un commit déjà poussé se fait aussi par un nouveau commit. L’historique reste lisible.

## 15. Aide-mémoire

| Je veux | Terminal | GitHub Desktop | Site |
| --- | --- | --- | --- |
| Copier un projet | `git clone adresse` | File, Clone repository | Bouton Code |
| Voir ce qui a changé | `git status` | Onglet Changes | — |
| Enregistrer | `git add` puis `git commit -m "..."` | Summary, Commit to branche | Edit, Commit changes |
| Envoyer | `git push` | Push origin | Déjà en ligne si le commit est fait sur le site |
| Recevoir | `git pull` | Fetch origin, Pull origin | La page du dépôt se met à jour toute seule |
| Nouvelle branche | `git switch -c nom` | Current Branch, New Branch | Branch selector, Create branch |
| Proposer la fusion | pousser la branche, puis ouvrir la demande | Preview Pull Request | Pull requests ou Merge requests, New |
| Historique | `git log --oneline` | History | Commits |

## 16. Problèmes fréquents

**`Permission denied (publickey)`.** La clé publique n’est pas sur le bon compte, ou vous clonez en SSH sans clé. Reprenez la section 5. GitLab détaille ce message : [Troubleshooting SSH](https://docs.gitlab.com/user/ssh_troubleshooting/).

**Le push demande un mot de passe en HTTPS.** Le mot de passe du compte ne marche pas. Passez en SSH, ou créez un jeton d’accès et utilisez-le comme mot de passe.

**`rejected`, `fetch first`.** Le site a des commits que vous n’avez pas. `git pull`, réglez un conflit s’il y en a un, puis `git push`.

**`main` et `master`.** Les dépôts récents utilisent `main`. Si le dépôt affiche `master`, gardez ce nom dans vos commandes. `git branch` montre la branche active avec une étoile.

**Deux README et des historiques sans lien.** Cela arrive si vous créez un dépôt GitHub avec un README, et que vous faites aussi `git init` dans un dossier déjà rempli, puis un push. Pour un dossier existant, créez le dépôt en ligne vide. Pour un projet neuf, créez-le sur le site avec un README, puis clonez-le.

**GitHub Desktop ne propose pas de merge request GitLab.** Normal. Poussez avec Desktop, puis ouvrez la merge request sur GitLab.

**Le fichier est ignoré alors que vous voulez l’envoyer.** Il correspond à une ligne du `.gitignore`, ou il a été ignoré plus tôt. `git status` ne le montre pas. Vérifiez `.gitignore` avant de forcer l’ajout.

## 17. Liens officiels

### GitHub

- [Documentation GitHub Desktop](https://docs.github.com/en/desktop)
- [Installer et configurer Git](https://docs.github.com/en/get-started/git-basics/set-up-git)
- [Créer un dépôt](https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories)
- [Cloner](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository)
- [Clé SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [Pull requests](https://docs.github.com/en/pull-requests/get-started/pull-request-quickstart)
- [Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects)
- [Issues](https://docs.github.com/en/issues/tracking-your-work-with-issues/using-issues/creating-an-issue)
- [Forks](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo)
- [Fichiers ignorés](https://docs.github.com/en/get-started/git-basics/ignoring-files)

### GitHub Desktop

- [Téléchargement](https://desktop.github.com/)
- [Premiers pas](https://docs.github.com/en/desktop/overview/getting-started-with-github-desktop)
- [Créer un dépôt](https://docs.github.com/en/desktop/overview/creating-your-first-repository-using-github-desktop)
- [Cloner](https://docs.github.com/en/desktop/adding-and-cloning-repositories/cloning-and-forking-repositories-from-github-desktop)
- [Ajouter un dossier existant](https://docs.github.com/en/desktop/adding-and-cloning-repositories/adding-an-existing-project-to-github-using-github-desktop)
- [Commit](https://docs.github.com/en/desktop/contributing-and-collaborating-using-github-desktop/making-changes-in-a-branch/committing-and-reviewing-changes-to-your-project)
- [Push](https://docs.github.com/en/desktop/contributing-and-collaborating-using-github-desktop/making-changes-in-a-branch/pushing-changes-to-github-from-github-desktop)
- [Mettre à jour la branche](https://docs.github.com/en/desktop/working-with-your-remote-repository-on-github-or-github-enterprise/syncing-your-branch-in-github-desktop)
- [Branches](https://docs.github.com/en/desktop/making-changes-in-a-branch/managing-branches-in-github-desktop)
- [Issue et pull request](https://docs.github.com/en/desktop/working-with-your-remote-repository-on-github-or-github-enterprise/creating-an-issue-or-pull-request-from-github-desktop)

### GitLab

- [Premiers pas avec Git](https://docs.gitlab.com/topics/git/get_started/)
- [Créer un projet](https://docs.gitlab.com/user/project/)
- [Premier commit](https://docs.gitlab.com/tutorials/make_first_git_commit/)
- [Cloner](https://docs.gitlab.com/topics/git/clone/)
- [Commit et push](https://docs.gitlab.com/topics/git/commit/)
- [Clés SSH](https://docs.gitlab.com/user/ssh/)
- [Merge requests](https://docs.gitlab.com/user/project/merge_requests/creating_merge_requests/)
- [Issues](https://docs.gitlab.com/user/project/issues/)
- [Tableaux d’issues](https://docs.gitlab.com/user/project/issue_board/)
- [Groupes](https://docs.gitlab.com/user/group/)
- [Éditeur web](https://docs.gitlab.com/user/project/repository/web_editor/)

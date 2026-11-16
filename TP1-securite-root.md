# TP1 - Sécurité Root

> Restreindre la connexion directe au compte root et configurer des privilèges sudo limités pour un utilisateur dédié.

## 🎯 Objectifs

1. Empêcher la connexion directe en tant que root
2. Créer un utilisateur avec des privilèges limités
3. Configurer sudo pour n'autoriser que des commandes précises

## Étape 1 - Désactiver la connexion root via SSH

```bash
sudo nano /etc/ssh/sshd_config
```

Modifier la ligne :
```
# Avant
#PermitRootLogin prohibit-password

# Après
PermitRootLogin no
```

Redémarrer le service :
```bash
sudo systemctl restart sshd
```

## Étape 2 - Créer un utilisateur non-root

```bash
sudo adduser utilisateur1
```

## Étape 3 - Ajouter l'utilisateur au groupe sudo

```bash
# Debian/Ubuntu
sudo usermod -aG sudo utilisateur1

# Red Hat/CentOS
sudo usermod -aG wheel utilisateur1
```

## Étape 4 - Configurer les permissions sudo précises

```bash
sudo visudo
```

Quelques options possibles à ajouter en fin de fichier :

```
# Autoriser uniquement apt update/upgrade sans mot de passe
utilisateur1 ALL=(ALL) NOPASSWD: /usr/bin/apt update

# Autoriser toutes les commandes avec mot de passe
utilisateur1 ALL=(ALL:ALL) ALL

# Autoriser toutes les commandes sans mot de passe (non recommandé)
utilisateur1 ALL=(ALL) NOPASSWD: ALL
```

> ⚠️ Toujours utiliser `visudo` pour éditer le fichier sudoers : il vérifie la syntaxe avant de sauvegarder.

## Étape 5 - Tester la configuration

```bash
su - utilisateur1
sudo apt update              # commande autorisée
sudo systemctl restart sshd  # commande non autorisée -> erreur attendue
```

## Étape 6 - Vérifier les restrictions

```bash
sudo -l -U utilisateur1
```

## 📎 Annexe

```bash
# Vérifier que le serveur SSH est installé
dpkg -l | grep openssh-server

# Sinon l'installer
sudo apt update
sudo apt install openssh-server
sudo systemctl restart ssh
```

## ✅ Conclusion

Ces étapes appliquent le **principe du moindre privilège** : limiter l'accès direct au compte root, créer un utilisateur dédié pour les tâches administratives, et restreindre précisément ce qu'il peut exécuter en sudo, réduisant ainsi la surface d'attaque.

## 🤔 Questions de réflexion

**Pourquoi désactiver le login root direct ?**
Cela réduit les risques d'erreurs et de compromission, et un compte utilisateur "standard" permet une traçabilité des commandes exécutées.

**Avantages du fichier sudoers pour gérer les privilèges ?**
Il permet de donner des droits limités à certains utilisateurs et facilite le suivi des commandes effectuées.

**Comment renforcer davantage la sécurité via sudoers ?**
Limiter l'accès à certaines commandes précises, et exiger un mot de passe obligatoire pour les commandes sensibles.

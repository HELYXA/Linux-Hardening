# TP2 — Vecteurs d'attaques

> Mettre en évidence les vecteurs d'attaque liés aux privilèges (mauvaises permissions, sudo mal configuré, scripts dangereux), puis les corriger.

## 1. Compte inutilement privilégié

**Erreur simulée** : créer un utilisateur et lui donner sudo sans raison.

```bash
sudo adduser testuser
sudo usermod -aG sudo testuser   # erreur : droits sudo complets inutiles
```

**Correction** :

```bash
sudo deluser testuser sudo
```

## 2. Permissions dangereuses sur un fichier sensible

**Erreur simulée** : rendre `/etc/shadow` lisible par tous.

```bash
sudo chmod 644 /etc/shadow
```

**Conséquence** : n'importe quel utilisateur peut lire les hachages de mots de passe → attaque par brute-force / cracking offline.

```bash
ls -l /etc/shadow   # vérifier l'exposition
```

**Correction** :

```bash
sudo chmod 640 /etc/shadow
```

## 3. Script partagé avec `sudo` en dur

**Erreur simulée** : un script accessible à tous contenant des commandes sudo destructrices.

```bash
sudo nano /home/scripttest.sh
```
```bash
#!/bin/bash
#sudo rm -rf /important/data
#sudo cp /tmp/file /etc/
```

**Risque** : si un attaquant modifie ce script, il peut exécuter du code arbitraire en root, supprimer des données critiques ou installer une backdoor.

**Correction** — déplacer le script hors de portée, le sécuriser :

```bash
sudo mv scripttest.sh /usr/local/sbin/
sudo chown root:root /usr/local/sbin/scripttest.sh
sudo chmod 750 /usr/local/sbin/scripttest.sh
```

Bonnes pratiques associées :
- Ne jamais mettre `sudo` en dur dans un script partagé
- Le script doit être appelé via `sudo ./script.sh` plutôt que contenir sudo lui-même
- Préférer des règles sudoers précises (commande unique, pas `ALL`)

## 4. Identifier les fichiers dangereux (world-writable)

```bash
sudo find / -type f -perm -o+w 2>/dev/null
```

Un fichier accessible en écriture par "others" (ex. dans `/usr/local/bin`) peut permettre une élévation de privilèges.

## 📋 En résumé

| Erreur | Correction |
|---|---|
| Droits sudo inutiles | `sudo deluser testuser sudo` |
| Fichiers sensibles trop ouverts | `chmod 640 /etc/shadow`, `chmod 644 /etc/passwd` |
| Répertoires critiques | `chmod 755 /bin /usr/bin` |
| Scripts à risque | Identifier + supprimer/restreindre (`chmod 750` + propriétaire root) |

## ✅ Bonnes pratiques de correction

- **Principe du moindre privilège** : seuls les comptes nécessaires ont sudo
- **Permissions minimales** : pas de fichiers world-writable dans `/etc`, `/bin`, `/usr/bin`
- **Scripts sûrs** : pas de sudo en dur, pas de droits d'écriture pour "others"
- **Surveillance** : auditer régulièrement avec `find` et `ls -l`
- **Automatisation** : cron qui alerte en cas de changement de permissions sur des fichiers sensibles

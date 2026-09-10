# TP3 — PAM (Pluggable Authentication Modules)

> Configurer PAM pour renforcer l'authentification : complexité des mots de passe, verrouillage après échecs, restrictions d'accès et plages horaires.

## 🎯 Objectifs

1. Renforcer la complexité des mots de passe avec `pam_pwquality`
2. Limiter les tentatives de connexion avec `pam_faillock`
3. Restreindre l'accès à certains utilisateurs avec `pam_access`
4. Configurer des restrictions temporelles avec `pam_time`

> ⚠️ Une mauvaise configuration de PAM peut bloquer l'accès au système entier — toujours garder une session root ouverte en parallèle pour tester sans risque de se retrouver bloqué dehors.

## Étape 1 — Sauvegarder les fichiers de configuration

```bash
sudo cp /etc/pam.d/common-password /etc/pam.d/common-password.bak
sudo cp /etc/pam.d/common-auth /etc/pam.d/common-auth.bak
sudo cp /etc/security/access.conf /etc/security/access.conf.bak
sudo cp /etc/security/time.conf /etc/security/time.conf.bak
```

## Étape 2 — Complexité des mots de passe (`pam_pwquality`)

```bash
sudo apt update
sudo apt install libpam-pwquality -y
```

## Étape 3 — Limiter les tentatives de connexion (`pam_faillock`)

```bash
sudo apt install libpam-modules-bin -y
sudo nano /etc/pam.d/common-auth
```

Règles configurées :
- `deny=5` : verrouille le compte après 5 tentatives échouées
- `unlock_time=300` : déverrouillage automatique après 5 minutes

## Étape 4 — Restreindre l'accès (`pam_access`)

```bash
sudo nano /etc/pam.d/common-auth
```
Ajouter après `auth required pam_tally2.so` :
```
account required pam_access.so
```

Puis configurer les règles :
```bash
sudo nano /etc/security/access.conf
```
```
# Refuser l'accès à tous par défaut
- : ALL : ALL

# Autoriser l'utilisateur 'admin' depuis partout
+ : admin : ALL

# Autoriser le groupe 'sudo' depuis le réseau local
+ : (sudo) : 192.168.x.x/24
```

## Étape 5 — Restrictions temporelles (`pam_time`)

```bash
sudo nano /etc/pam.d/common-account
```
```
account required pam_time.so
```

```bash
sudo nano /etc/security/time.conf
```
```
# Autoriser le groupe 'employees' du lundi au vendredi de 8h à 18h
login ; * ; * ; Al0800-1800

# Refuser l'accès à l'utilisateur 'test' à tout moment
login ; * ; test ; !Al0000-2400
```

## Étape 6 — Restreindre `su` avec `pam_wheel`

`su` permet de changer d'identité (généralement vers root) sans se déconnecter. Sans restriction, n'importe quel utilisateur connaissant le mot de passe root peut l'utiliser.

```bash
# Vérifier / créer le groupe dédié
getent group | grep wheel
sudo groupadd su   # si besoin

# Restreindre l'accès à su via PAM (/etc/pam.d/su)
auth required pam_wheel.so use_uid
# ou pour un groupe personnalisé :
auth required pam_wheel.so use_uid group=su

# Ajouter les utilisateurs autorisés
sudo usermod -aG wheel nom_utilisateur

# Vérifier
groups nom_utilisateur
```

Résultat attendu : un utilisateur non membre du groupe obtient `su: Permission denied`.

## Étape 7 — Tester la configuration

```bash
passwd                # tester un mot de passe faible -> doit être refusé
# tenter 5+ connexions échouées -> compte verrouillé 5 min
```

## 🔁 Restauration en cas de problème

```bash
sudo cp /etc/pam.d/common-password.bak /etc/pam.d/common-password
sudo cp /etc/pam.d/common-auth.bak /etc/pam.d/common-auth
sudo cp /etc/security/access.conf.bak /etc/security/access.conf
sudo cp /etc/security/time.conf.bak /etc/security/time.conf
```

## ✅ Conclusion

PAM a été configuré pour exiger des mots de passe forts, limiter les tentatives de connexion, restreindre l'accès à certains utilisateurs/réseaux, appliquer des plages horaires, et restreindre `su` au groupe `wheel`.

## 🔗 Ressources

- [pam_faillock (man)](https://linux.die.net/man/8/pam_faillock)
- [pam_access (man)](https://linux.die.net/man/8/pam_access)
- [access.conf (man)](https://linux.die.net/man/5/access.conf)
- [pam_time (man)](https://linux.die.net/man/8/pam_time)
- [time.conf (man)](https://linux.die.net/man/5/time.conf)

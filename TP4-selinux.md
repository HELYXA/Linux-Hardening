# TP4 — SELinux

> Installer et activer SELinux sur une distribution qui utilise AppArmor par défaut (Ubuntu/Kali), puis explorer les contextes de sécurité et le mode enforcing.

## 📌 Contexte

Ubuntu 24.04 LTS (et les versions récentes) n'utilisent pas SELinux par défaut — le contrôle d'accès obligatoire (MAC) par défaut est **AppArmor**. Il reste possible d'activer SELinux manuellement.

## Installation

```bash
sudo apt update && sudo apt upgrade -y
cat /etc/os-release   # vérifier la version

# Installer les paquets SELinux
sudo apt install selinux-basics selinux-policy-default auditd
```
- `selinux-basics` : scripts de configuration de base
- `selinux-policy-default` : politiques de sécurité par défaut
- `auditd` : daemon d'audit de sécurité

```bash
# Désactiver AppArmor (les deux MAC ne coexistent pas)
sudo systemctl stop apparmor
sudo systemctl disable apparmor

# Activer SELinux et redémarrer
sudo selinux-activate
sudo reboot

# Vérifier l'état après redémarrage
sestatus     # SELinux status: enabled
getenforce   # Current mode: Permissive
```

Commandes utiles :
```bash
sudo chcon -t httpd_sys_content_t /var/www/html/example.txt
sudo ausearch -m avc -ts today
```

## Exercice 1 — Contextes de sécurité

```bash
id                    # contexte SELinux de l'utilisateur
ps axZ | less          # contexte SELinux de chaque processus
ls -lZ                  # contexte SELinux des fichiers (home, /bin, /tmp...)
```

**Observations** :
- Processus système root → label `system_r`
- Processus système utilisateur → `unconfined_u:unconfined_r:unconfined_t`
- Fichiers "classiques" du home → `system_u:object_r:user_home_t:s0`
- Répertoires XDG (Documents, Downloads...) → `system_u:object_r:xdg_*_t:s0`

Copier un fichier en préservant son contexte de sécurité :
```bash
cp --preserve=context ~/Desktop/test1.txt /tmp/
```

Changer le contexte de sécurité d'un fichier :
```bash
sudo chcon -t tmp_t /tmp/test1.txt
```
Contexte résultant : `user: unconfined_u`, `rôle: object_r`, `type: tmp_t`, `level: s0`.

## Exercice 2 — Contextes des applications (Apache) et mode enforcing

```bash
# S'assurer que SELinux est bien installé et actif
sudo apt install selinux selinux-utils selinux-policy-default policycoreutils -y
sestatus
getenforce

# Activer SELinux au niveau GRUB
sudo nano /etc/default/grub
# GRUB_CMDLINE_LINUX="selinux=1 security=selinux"
sudo update-grub
sudo reboot
```

Analyser les opérations refusées (AVC) :
```bash
sudo ausearch -m avc -ts today
sudo ausearch -m avc -ts today | sudo audit2allow -a -w
```

Générer une politique personnalisée à partir des refus :
```bash
sudo ausearch -m avc -ts today | sudo audit2allow -M custom_policy
sudo semodule -i custom_policy.pp
sudo semodule -l | grep custom_policy   # vérifier l'installation
```

### Test concret avec Apache

```bash
# Créer un fichier test dans Documents
touch ~/Documents/test2
ls -lZ ~/Documents/test2

# Le copier dans /var/www/html en préservant le contexte
sudo cp --preserve=context ~/Documents/test2 /var/www/html/
ls -lZ /var/www/html/test2

# Tester le téléchargement
wget http://localhost/test2
```

En mode **Permissive**, l'accès n'est pas bloqué. Passage en mode **Enforcing** :
```bash
sudo setenforce 1
wget http://localhost/test2   # -> 403 Forbidden
sudo ausearch -m avc -ts today
```

**Lecture d'un refus AVC** :
```
avc: denied { getattr } :: SELinux a refusé l'accès
comm="apache2"                                  # processus concerné
path="/var/www/html/test2"                       # fichier concerné
scontext=system_u:system_r:httpd_t:s0             # contexte du processus Apache
tcontext=unconfined_u:object_r:user_home_t:s0     # contexte du fichier (mauvais type)
tclass=file
permissive=0                                      # SELinux bloque effectivement
```

Le fichier avait le mauvais contexte (`user_home_t` au lieu de `httpd_sys_content_t`), d'où le refus par Apache.

Remettre le contexte d'origine et observer l'effet de `unconfined_exec_t` sur Apache :
```bash
sudo restorecon -v /var/www/html/test2
sudo systemctl stop apache2
sudo chcon -t unconfined_exec_t /usr/sbin/apache2
sudo systemctl start apache2
ps -eZ | grep apache2
wget http://localhost/test2   # -> 200 OK
```

Modifier le contexte de l'exécutable Apache en `unconfined_exec_t` le rend "non restreint" par SELinux — il peut alors accéder aux fichiers même avec un contexte incorrect (ce qui illustre bien le risque de mal configurer les contextes).

## Exercice 3 — User mapping et `semanage`

```bash
sudo semanage login -l     # user mapping
sudo semanage user -l      # mappings de rôles utilisateur
```

Observation : `__default__` et `root` → `unconfined_u` (rôles `system_r`, `unconfined_r`).

```bash
# Ajouter un compte de test et l'observer
sudo useradd aleatoire
sudo passwd aleatoire
```

Le fichier `/proc/self/attr/prev` contient le contexte SELinux du processus parent qui a lancé le shell actuel — utile pour tracer une chaîne de connexions (ex. via SSH).

## 🛠️ Stack / outils

`SELinux` `AppArmor` `auditd` `audit2allow` `semanage` `chcon` `restorecon` `GRUB`

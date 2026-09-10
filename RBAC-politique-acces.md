# TP - Politique de contrôle d'accès basée sur les rôles (RBAC)

> Document officiel de politique RBAC pour un environnement Linux : rôles, gestion des comptes, processus de validation, audit.

**Contexte** : cours Sécurité Linux, Bachelor 2 Cybersécurité - rédaction d'un document de politique d'accès aussi rigoureux qu'un vrai document d'entreprise.

## Objet

Cette politique définit les principes et procédures de mise en œuvre d'un modèle **RBAC (Role-Based Access Control)** dans un environnement Linux, afin d'assurer une gestion sécurisée des accès en réduisant les risques de compromission par une attribution précise des privilèges selon le rôle de chaque utilisateur.

Elle s'inscrit dans la protection des données sensibles, la conformité aux normes de cybersécurité (ex. ISO 27001) et la prévention des accès non autorisés, en s'appuyant sur les mécanismes natifs de Linux (groups, sudo, PAM...).

## Principes généraux

- **Séparation des rôles** : chaque utilisateur est assigné à un rôle prédéfini, limitant ses actions aux tâches nécessaires
- **Moindre privilège** : chaque rôle ne reçoit que les permissions minimales requises
- **Hiérarchie des rôles** : héritage possible (un admin hérite des droits d'un user standard)
- **Contrôle centralisé** : gestion via `/etc/group`, `/etc/sudoers`, modules PAM
- **Authentification & autorisation** : authentification forte (mot de passe complexe, clé SSH, MFA) + vérification des rôles via SELinux/AppArmor

## Rôles et responsabilités

| Rôle | Responsabilités | Permissions | Exemple d'assignation |
|---|---|---|---|
| **Utilisateur standard** | Utiliser les applications autorisées sans modifier la config système | Pas de sudo, accès aux groupes de base | `usermod -aG users username` |
| **Administrateur système** | Maintenir la sécurité, appliquer les mises à jour, surveiller les logs | Sudo complet, groupe wheel/root | `usermod -aG wheel username` + `visudo` |
| **Auditeur** | Vérifier la conformité, signaler les anomalies | Lecture seule des logs (`journalctl`, `ausearch`), pas de modification | `username ALL=(ALL) NOPASSWD: /usr/bin/journalctl` |
| **Superviseur** | Valider les demandes d'accès, révoquer les comptes suspects | Administration étendue, gestion des rôles | `usermod -aG sudo username` |

## Gestion des comptes

- **Création** : `useradd -m -G users username`, rôles assignés via `usermod -aG groupname username`
- **Modification** : `usermod` ou édition de `/etc/group` ; mots de passe gérés avec `passwd` (min. 12 caractères, complexité requise)
  - `passwd -l username` : verrouiller un compte
  - `chpasswd` : changement en masse via fichier
- **Suppression** : `userdel -r username` ; comptes inactifs 90+ jours désactivés automatiquement via cron
  ```bash
  # /etc/crontab
  0 2 * * * root find /home -type d -mtime +90 -exec userdel -r {} \;
  ```
- **Expiration** : `chage -E 2025-12-31 username`
- **Stockage** : `/etc/passwd`, `/etc/shadow`, `/etc/group` (mots de passe chiffrés SHA-512)
- Tous les changements sont journalisés dans `/var/log/auth.log`

## Processus de demande et validation

1. **Demande** : formulaire officiel (ticket / email) incluant justification, rôle souhaité, durée
2. **Validation** : vérification du principe du moindre privilège par l'administrateur ; approbation à deux niveaux pour les rôles élevés
   - `id username` : groupes de l'utilisateur
   - `sudo -l -U username` : permissions sudo
3. **Approbation** : configuration via `usermod -aG newgroup username` + `service ssh reload`
4. **Refus** : explication fournie, demande révisable
5. **Délai** : traitement sous 48h ouvrables

## Audit et contrôle

- **Surveillance continue** : `auditd`
  ```bash
  auditctl -a always,exit -F arch=b64 -S execve
  ausearch -m USER_AUTH
  ```
- **Contrôles périodiques** : `getent group`, `sudo -l` (mensuel)
- **Détection d'anomalies** : SELinux + `setroubleshoot`
  ```bash
  sestatus
  sealert -a /var/log/audit/audit.log
  ```
- **Rapports trimestriels** :
  ```bash
  journalctl --since "2025-01-01" --until "2025-03-31" > rapport_trimestre1.log
  ```
- **Réponse aux incidents** : révocation immédiate + investigation
  ```bash
  usermod -L username   # verrouille le compte
  last username          # dernières connexions
  ```

## Revue et amélioration

- Révision annuelle ou après incident majeur
- Amélioration continue : intégration des nouvelles menaces (ex. élévation de privilèges)
- Formation de sensibilisation annuelle sur RBAC et bonnes pratiques
- Suivi de métriques (ex. nombre d'incidents d'accès non autorisé, objectif de réduction annuel)

---

*Document rédigé dans le cadre du Bachelor 2 Cybersécurité (NEXA Digital School) - module Sécurité Linux.*

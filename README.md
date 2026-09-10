# 🐧 Linux Hardening — Travaux Pratiques

> Série de 5 TP sur le durcissement (hardening) d'un système Linux : gestion des privilèges, vecteurs d'attaque, PAM, SELinux, et politique d'accès RBAC.

## 📚 Sommaire

| TP | Sujet |
|---|---|
| [TP1 — Sécurité Root](TP1-securite-root.md) | Désactivation du login root, création d'un utilisateur avec sudo restreint |
| [TP2 — Vecteurs d'attaques](TP2-vecteurs-attaques.md) | Comptes sur-privilégiés, permissions dangereuses, scripts à risque |
| [TP3 — PAM](TP3-pam.md) | Complexité des mots de passe, anti-bruteforce, restrictions d'accès et horaires |
| [TP4 — SELinux](TP4-selinux.md) | Installation, contextes de sécurité, mode enforcing, analyse des AVC |
| [Politique RBAC](RBAC-politique-acces.md) | Document officiel de contrôle d'accès basé sur les rôles |

## 🎯 Ce que couvre cette série de TP

- **Principe du moindre privilège** appliqué concrètement (sudo, permissions fichiers, groupes)
- **Identification et correction** de vecteurs d'attaque courants (mauvaises permissions, scripts avec sudo en dur)
- **Authentification renforcée** via PAM (pam_pwquality, pam_faillock, pam_access, pam_time, pam_wheel)
- **Contrôle d'accès obligatoire (MAC)** avec SELinux : contextes de sécurité, mode permissive/enforcing, résolution de refus AVC sur un serveur Apache
- **Formalisation** d'une politique de sécurité RBAC complète, comme dans un contexte professionnel

## 🛠️ Environnement

`Ubuntu 24.04 LTS` `Kali Linux` `SSH` `sudo / visudo` `PAM` `SELinux` `AppArmor` `auditd`

---

*Travaux pratiques réalisés dans le cadre du Bachelor 2 Cybersécurité (NEXA Digital School) — module Sécurité Linux.*

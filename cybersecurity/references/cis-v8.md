# CIS Controls v8 — Reference

> Charger ce fichier quand : l'utilisateur mentionne "CIS", "CIS Controls", "CIS Benchmarks",
> "hardening", "baseline sécurité", "IG1/IG2/IG3", ou quand une évaluation de maturité
> opérationnelle est demandée. Aussi pertinent pour les PME qui veulent une roadmap pragmatique.

---

## Vue d'ensemble CIS Controls v8

Les 18 contrôles CIS sont organisés en **3 groupes d'implémentation (IG)** selon la maturité :

```
IG1 — Hygiène de base (toutes organisations)         → 56 safeguards
IG2 — Organisations avec équipe IT dédiée            → +74 safeguards (= 130 total)
IG3 — Organisations avec experts sécurité            → +23 safeguards (= 153 total)
```

> **Analogie :** IG1 = ceinture de sécurité. IG2 = airbags. IG3 = cage de sécurité complète.
> Commencer par IG1 avant de viser IG3.

---

## Les 18 contrôles — Mapping avec le skill

### CIS 1 — Inventory and Control of Enterprise Assets
**IG1 minimum :**
- Inventaire de tous les assets (serveurs, VMs, conteneurs, devices)
- Découverte active des assets non autorisés
- Mise à jour de l'inventaire sous 1 semaine

**Lien skill :** Connaître son périmètre avant d'auditer. Demander systématiquement la cartographie des composants.

---

### CIS 2 — Inventory and Control of Software Assets
**IG1 minimum :**
- Inventaire de tous les logiciels autorisés
- Blocage des logiciels non autorisés (application whitelisting si possible)
- Suppression des logiciels non utilisés

**Lien skill :** → **Couche 2 (Dépendances)** — scanner `pom.xml`, `package.json`, etc.

---

### CIS 3 — Data Protection
**IG1 minimum :**
- Inventaire des données sensibles
- Chiffrement des supports amovibles
- Accès aux données selon le besoin

**IG2 :**
- Classification des données (4 niveaux minimum)
- DLP déployé sur les canaux de sortie
- Chiffrement au repos des données sensibles

**IG3 :**
- Chiffrement de bout en bout
- Gestion centralisée des clés (HSM ou Vault)

**Lien skill :** → **Couche 3 (Secrets)** + ISO A.8.10-A.8.12

---

### CIS 4 — Secure Configuration of Enterprise Assets and Software
**C'est le contrôle le plus directement lié à la Couche 4 du skill.**

**IG1 minimum :**
- Changer les configurations par défaut (mots de passe, ports, services)
- Désactiver les services inutiles
- Appliquer les mises à jour de sécurité sous 1 mois (CRITIQUE : 14 jours)

**IG2 :**
- Baseline documentée pour chaque type d'asset
- Gestion de configuration automatisée (Ansible, Chef, Puppet)
- Scan de conformité régulier (écart à la baseline)

**IG3 :**
- Dérive de configuration détectée en temps réel
- Configuration immutable (GitOps, IaC uniquement)

**Benchmarks CIS disponibles (références) :**
- CIS Docker Benchmark
- CIS Kubernetes Benchmark
- CIS Ubuntu Linux Benchmark
- CIS NGINX Benchmark
- CIS PostgreSQL Benchmark

```bash
# Exemple : vérifier conformité Docker avec docker-bench-security
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  docker/docker-bench-security
```

---

### CIS 5 — Account Management
**IG1 minimum :**
- Inventaire de tous les comptes (actifs et inactifs)
- Désactiver les comptes après X jours d'inactivité (30-90j selon politique)
- Supprimer les comptes des anciens employés sous 24h

**IG2 :**
- Revue trimestrielle des droits d'accès
- Comptes de service avec credentials rotatifs
- Pas de comptes partagés (un humain = un compte)

**Lien skill :** → **Couche 5 (Auth/Authz)**

---

### CIS 6 — Access Control Management
**IG1 minimum :**
- Principe du moindre privilège appliqué
- MFA sur tous les accès à distance
- MFA sur les comptes admin

**IG2 :**
- Just-in-time access (accès temporaires à la demande)
- Revue des accès privilegiés mensuelle
- PAM (Privileged Access Management) déployé

**IG3 :**
- Zero standing privileges (aucun accès permanent en production)
- Session recording pour les accès admin

**Checklist rapide :**
```
[ ] MFA activé pour tous les comptes (pas juste les admins)
[ ] Pas de comptes avec droits permanents en production
[ ] Revue des droits documentée et tracée
[ ] Politique de mot de passe conforme (longueur ≥ 12, pas de rotation forcée périodique)
[ ] Gestionnaire de mots de passe déployé
```

---

### CIS 7 — Continuous Vulnerability Management
**IG1 minimum :**
- Scan de vulnérabilités automatisé (hebdomadaire minimum)
- Processus de correction avec SLA

**IG2 :**
- Scan authentifié (plus complet)
- Scan DAST sur les applications web
- Corrélation avec la threat intelligence

**IG3 :**
- Programme de Bug Bounty
- Pentest trimestriel
- Red team annuel

**SLA de correction CIS recommandés :**
| CVSS | Délai max |
|------|-----------|
| 9.0-10.0 (Critique) | 15 jours |
| 7.0-8.9 (Haute) | 30 jours |
| 4.0-6.9 (Moyenne) | 60 jours |
| 0.1-3.9 (Basse) | 90 jours |

---

### CIS 8 — Audit Log Management
→ **Couche 7 du skill (Observabilité)** — référence principale.

**IG1 minimum :**
- Logs activés sur tous les assets critiques
- Horodatage NTP synchronisé
- Rétention minimum 90 jours

**IG2 :**
- Centralisation des logs (SIEM)
- Alertes sur événements critiques
- Rétention 13 mois minimum

**IG3 :**
- Corrélation automatisée des événements
- Threat hunting proactif sur les logs
- Logs immuables (write-once)

---

### CIS 9 — Email and Web Browser Protections
- Anti-phishing, anti-spam, DMARC/SPF/DKIM
- Filtrage DNS (blocage domaines malveillants)
- Browser extensions sous contrôle

**Configuration email (vérifier) :**
```dns
# SPF
v=spf1 include:_spf.google.com ~all

# DMARC
v=DMARC1; p=quarantine; rua=mailto:dmarc@mondomaine.com

# DKIM : clé configurée chez le provider email
```

---

### CIS 10 — Malware Defenses
- EDR/Antivirus sur tous les endpoints
- Scan des emails et pièces jointes
- Application allowlisting si possible

---

### CIS 11 — Data Recovery
- Sauvegardes automatisées pour tous les assets critiques
- Règle 3-2-1 : 3 copies, 2 supports différents, 1 hors site
- Test de restauration documenté et exécuté trimestriellement

```
3-2-1-1-0 (version renforcée) :
3 copies
2 supports différents
1 hors site
1 copie offline (air-gapped)
0 erreur vérifiée au dernier test de restauration
```

---

### CIS 12 — Network Infrastructure Management
→ **Couche 4 du skill** (Infrastructure réseau)

- Diagramme réseau à jour
- Filtrage du trafic aux frontières (firewall)
- Segmentation réseau (VLAN, zones)
- Gestion des équipements réseau (authentification, mises à jour)

---

### CIS 13 — Network Monitoring and Defense
- NDR (Network Detection and Response) ou IDS/IPS
- Monitoring du trafic DNS (détection C2)
- Filtrage du trafic sortant (pas seulement entrant)

---

### CIS 14 — Security Awareness and Skills Training
- Formation sécurité pour tous les employés (annuelle minimum)
- Phishing simulé
- Formation spécifique développeurs (OWASP, Secure Coding)
- Sensibilisation des managers aux risques cyber

---

### CIS 15 — Service Provider Management
- Inventaire des fournisseurs avec accès aux données
- Évaluation de sécurité annuelle des fournisseurs critiques
- Clauses contractuelles de sécurité
- Droits d'audit contractualisés

---

### CIS 16 — Application Software Security
**C'est le contrôle le plus centré sur le développement.**
→ Correspond aux **Couches 1 à 6 du skill** dans leur globalité.

**IG2 minimum :**
- [ ] Processus SDLC sécurisé documenté
- [ ] SAST intégré au pipeline CI/CD
- [ ] Revue de code de sécurité pour les changements critiques
- [ ] WAF déployé devant les applications web exposées
- [ ] Gestion des secrets via secret manager

**IG3 :**
- [ ] DAST automatisé pré-release
- [ ] IAST en environnement de staging
- [ ] Pentest avant chaque release majeure
- [ ] Bug bounty program

---

### CIS 17 — Incident Response Management
→ **Lien NIST CSF RESPOND/RECOVER**

- Plan de réponse à incident documenté et approuvé
- Équipe IR identifiée avec rôles clairs
- Exercices de simulation (tabletop) annuels
- Processus de notification légale (RGPD 72h, ANSSI)

---

### CIS 18 — Penetration Testing
- Pentest externe annuel minimum (IG2)
- Scope documenté et approuvé
- Rapport formel avec findings et remédiation
- Retest post-correction
- Red team annuel (IG3)

---

## Rapport de conformité CIS v8 — Template

```markdown
# Rapport CIS Controls v8
**Date** : [date]
**Groupe d'implémentation cible** : IG[1/2/3]
**Score actuel** : XX/153 safeguards (XX%)

## État par contrôle

| CIS | Nom | IG | Safeguards | Conformes | % | Priorité |
|-----|-----|----|-----------:|----------:|---|---------|
| 1  | Asset Inventory | IG1 | 5 | 3 | 60% | 🔴 |
| 2  | Software Inventory | IG1 | 7 | 5 | 71% | 🟠 |
| 3  | Data Protection | IG1 | 14 | 8 | 57% | 🔴 |
| 4  | Secure Config | IG1 | 12 | 9 | 75% | 🟡 |
| 5  | Account Mgmt | IG1 | 6 | 5 | 83% | 🟢 |
| 6  | Access Control | IG1 | 8 | 6 | 75% | 🟡 |
| 7  | Vuln Management | IG1 | 7 | 4 | 57% | 🔴 |
| 8  | Audit Logs | IG1 | 5 | 4 | 80% | 🟢 |
| 16 | App Security | IG2 | 14 | 7 | 50% | 🔴 |
| 18 | Pentest | IG2 | 5 | 2 | 40% | 🔴 |

## Top 5 actions prioritaires IG1
[Liste avec responsable, deadline, effort estimé]

## Roadmap IG1 → IG2
[Plan trimestriel]
```

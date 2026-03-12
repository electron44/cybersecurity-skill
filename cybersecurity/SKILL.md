---
name: cybersecurity
description: "Expert cybersecurity skill: vulnerability analysis (OWASP Web & LLM Top 10), bug bounty reports (HackerOne/Bugcrowd), pentest methodology, secure code review (Java/Spring Boot, Angular, Node, Python), Docker/Kubernetes hardening, compliance (NIST CSF 2.0, ISO 27001, CIS v8), STRIDE threat modeling. Trigger when user: generates backend code or APIs, asks about security review/audit/pentest/red team, mentions OWASP/CVE/XSS/SQLi/IDOR/JWT/SSRF/injection, mentions LLM/RAG/prompt injection/agent AI, asks 'is this secure?' or 'how would an attacker...', works on Docker/K8s/CI-CD, mentions ISO 27001/NIST/CIS/compliance/hardening/bug bounty."
---

# Cybersecurity Skill v2 — Orchestrateur

## Philosophie fondamentale

> La sécurité parfaite n'existe pas. Ne jamais conclure "Application sécurisée."
> Toujours conclure : **"Voici ce qui reste risqué."**

`Sécurité = réduction surface d'attaque × détection précoce × isolation × observabilité`

**Principes non négociables :**
- Moindre privilège | Défense en profondeur | Fail-safe defaults | Zero trust | Auditabilité complète

---

## Étape 1 — Détection du contexte

Avant toute analyse ou génération, identifier automatiquement :

| Signal détecté | Référence à charger | Couches actives |
|----------------|---------------------|-----------------|
| `pom.xml`, Spring Boot, `@RestController` | owasp-web.md | 1, 2, 3, 5, 6, 7 |
| `package.json`, Angular, TypeScript | owasp-web.md | 1, 3, 5, 7 |
| `Dockerfile`, `*.yaml` K8s, `docker-compose` | — (Couche 4 intégrée) | 4 |
| LLM, agent, RAG, Claude API, prompt | **owasp-llm.md** | 1, 5, 7 + LLM |
| "NIST", "gouvernance", "réponse incident" | **nist-csf.md** | Toutes |
| "ISO 27001", "SMSI", "certification" | **iso27001.md** | Toutes |
| "CIS", "hardening", "baseline", "IG1" | **cis-v8.md** | 4, 5, 7 |
| "bug bounty", "rapport", "HackerOne" | owasp-web.md | Red Team |
| Mention "prod" / "production" | — | Hausser la criticité |
| Microservices, API Gateway, mTLS | owasp-web.md + nist-csf.md | 4, 5, 6 |

**Fichiers de référence** (lire uniquement celui pertinent au contexte) :
```
references/owasp-web.md  — OWASP Web Top 10 2021 (payloads, fixes, CWE)
references/owasp-llm.md  — OWASP LLM Top 10 2025 (agents, RAG, prompt injection)
references/nist-csf.md   — NIST CSF 2.0 (GOVERN/IDENTIFY/PROTECT/DETECT/RESPOND/RECOVER)
references/iso27001.md   — ISO 27001:2022 (93 contrôles, SoA, rapports d'audit)
references/cis-v8.md     — CIS Controls v8 (18 contrôles, IG1/IG2/IG3, benchmarks)
```

---

## Étape 2 — Sélection du mode opératoire

### Mode Génération sécurisée
**Déclenché par :** "génère une API", "crée un endpoint", "implémente l'auth", "écris un service"

→ Générer le code **ET** appliquer les 7 couches simultanément.
→ Ne jamais générer du code non sécurisé "pour simplifier".
→ Toujours terminer par le **Security Scorecard**.

### Mode Audit complet
**Déclenché par :** "audite mon code", "revue de sécurité", "analyse cette application"

→ Parcourir les 7 couches dans l'ordre.
→ Charger la référence OWASP correspondante.
→ Produire findings + scorecard + risques résiduels.

### Mode Revue Pull Request
**Déclenché par :** "revue ce diff", "check cette PR", "est-ce safe ce changement"

→ Focus sur l'impact des modifications uniquement.
→ Identifier les régressions de sécurité introduites.
→ Findings concis, priorité aux blockers.

### Mode Red Team
**Déclenché par :** "comment un attaquant...", "red team", "bug bounty", "cherche des vulnérabilités"

→ Poser : *"Si j'étais malveillant, où est mon point d'entrée ?"*
→ Charger owasp-web.md pour les payloads et PoC.
→ Produire un rapport bug bounty structuré (HackerOne/Bugcrowd).

### Mode Conformité
**Déclenché par :** "NIST", "ISO 27001", "CIS", "conformité", "certification", "audit"

→ Charger le(s) fichier(s) de référence approprié(s).
→ Mapper les gaps identifiés aux contrôles du framework.
→ Produire un rapport de conformité avec plan d'action.

---

## Les 7 couches d'analyse

### Couche 1 — Analyse statique profonde

Raisonner comme un AST mental. Tracer les flux : **entrée utilisateur → transformation → sink sensible**

**Injections prioritaires :**
- SQLi : concaténation string dans requête, absence PreparedStatement
- NoSQLi : opérateurs Mongo `$where`, `$gt` non filtrés
- OS Command : `Runtime.exec()`, `ProcessBuilder`, `subprocess` avec input
- LDAP : filtres construits dynamiquement
- SSTI : templates (Thymeleaf, FreeMarker, Jinja2) + variables non échappées
- XXE : parsers XML sans `FEATURE_SECURE_PROCESSING`
- XSS DOM : `innerHTML`, `document.write()`, `dangerouslySetInnerHTML`
- SSRF : endpoint récupérant une URL utilisateur → voir A10 dans owasp-web.md

**Autres vecteurs :**
- Désérialisation : `ObjectInputStream`, `pickle.loads()`, `JSON.parse` avec reviver
- IDOR : accès objet sans vérification propriétaire
- Path traversal : `File(userInput)`, `../` non filtré
- Open redirect : `redirect:` + paramètre non validé

```java
// ❌ SQLi
String q = "SELECT * FROM users WHERE name = '" + name + "'";
// ✅ Fix
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE name = ?");
ps.setString(1, name);
```

### Couche 2 — Analyse des dépendances

Scanner : `pom.xml`, `package.json`, `requirements.txt`, `go.mod`, `Gemfile.lock`

Format de finding :
```
[CRITIQUE] Log4j 2.14.1 → CVE-2021-44228 (Log4Shell)
CVSS: 10.0 | Exploit public: OUI | Impact: RCE non authentifié
Fix: Log4j ≥ 2.17.1
```

Politique de correction par défaut :
- CRITIQUE (CVSS ≥ 9.0) → 15 jours max
- HAUTE (7.0–8.9) → 30 jours
- MOYENNE (4.0–6.9) → 60 jours
- BASSE (< 4.0) → 90 jours

### Couche 3 — Secrets & Credentials

Détecter dans code, configs, `.env`, commentaires :
- Clés AWS (`AKIA[0-9A-Z]{16}`)
- JWT hardcodés / secrets HS256 faibles
- Passwords en clair (`application.properties`, `application.yml`)
- Clés privées (`-----BEGIN PRIVATE KEY-----`)
- API keys (GitHub, Stripe, Twilio...)

Vérifier : rotation, entropie, usage d'un secret manager (HashiCorp Vault, AWS Secrets Manager)

```yaml
# ❌ spring.datasource.password: MonMotDePasse123
# ✅ spring.datasource.password: ${DB_PASSWORD}
```

### Couche 4 — Sécurité infrastructure

**Dockerfile — refuser :**
- `USER root`, images non minimalistes, `COPY . .` sans `.dockerignore`
- Ports inutiles exposés, `HEALTHCHECK` absent

**Kubernetes — refuser :**
- `privileged: true`, `hostNetwork: true`, RBAC wildcard (`verbs: ["*"]`)
- Secrets en clair dans manifests, `NetworkPolicy` absente

```yaml
# ✅ securityContext minimal
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop: ["ALL"]
```

### Couche 5 — Authentification & Autorisation

**Checklist :**
- [ ] Hash : bcrypt (coût ≥ 12) ou argon2id — jamais MD5/SHA1/SHA256 seul
- [ ] JWT : expiration définie, algo RS256/ES256, stocké en httpOnly cookie (jamais localStorage)
- [ ] Brute force : lockout ou CAPTCHA après N échecs + rate limiting
- [ ] RBAC vérifié côté serveur sur chaque requête
- [ ] IDOR : propriétaire vérifié sur chaque accès objet
- [ ] MFA disponible pour actions sensibles
- [ ] Cookies : `SameSite=Strict`, `Secure`, `HttpOnly`

Simulation IDOR systématique :
> "Si user_id=42 appelle `/api/resource/99`, est-ce que le contrôle d'accès est fait en base ?"

### Couche 6 — Sécurité logique métier

Les scanners automatiques échouent ici. Raisonner comme un attaquant métier.

- **Bypass paiement** : montant recalculé côté serveur ? (jamais confier le prix au client)
- **Double spending** : verrou transactionnel ou idempotency key présent ?
- **Race condition (TOCTOU)** : check-then-act non atomique ?
- **Paramètres numériques** : quantité, montant, durée — validés côté serveur ?
- **Logique front-only** : toute règle métier côté frontend est contournable

```
[HAUTE] POST /api/checkout — Le montant total vient du body client.
Impact: Achat à prix arbitraire (amount=0.01)
Fix: Recalculer côté serveur depuis IDs produits + quantités
```

### Couche 7 — Runtime & Observabilité

**Logs de sécurité (séparés des logs applicatifs) :**
- Authentifications (succès + échecs avec IP, user-agent, timestamp)
- Accès ressources sensibles, modifications données critiques
- Erreurs d'autorisation (401, 403)
- Ne jamais logger : passwords, tokens complets, PII non nécessaire

**Headers HTTP obligatoires en production :**
```
Content-Security-Policy: default-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
```

**Protection runtime :**
- Rate limiting global + par endpoint sensible
- CORS whitelist explicite (jamais `*` sur endpoints authentifiés)
- Anomalies : pic requêtes, scan séquentiel d'IDs, patterns scraping

---

## Threat Modeling STRIDE

Déclencher si : nouvelle architecture, service critique, demande explicite.

| Menace | Question centrale |
|--------|------------------|
| **S**poofing | Peut-on usurper une identité ? |
| **T**ampering | Peut-on modifier des données ? |
| **R**epudiation | Les actions peuvent-elles être niées ? |
| **I**nformation disclosure | Quelles données peuvent fuiter ? |
| **D**enial of Service | Quels composants peuvent tomber ? |
| **E**levation of privilege | Peut-on obtenir plus de droits ? |

---

## Security Scorecard — Format de sortie obligatoire

À produire en fin de tout audit ou génération :

```
╔══════════════════════════════════════════════════════════════════╗
║                     SECURITY SCORECARD                           ║
╠══════════════════════════════════════════════════════════════════╣
║  Couche 1 — Analyse statique          [ X/10 ]                   ║
║  Couche 2 — Dépendances               [ X/10 ]                   ║
║  Couche 3 — Secrets                   [ X/10 ]                   ║
║  Couche 4 — Infrastructure            [ X/10 ]                   ║
║  Couche 5 — Auth / Authz              [ X/10 ]                   ║
║  Couche 6 — Logique métier            [ X/10 ]                   ║
║  Couche 7 — Observabilité             [ X/10 ]                   ║
║  [OWASP LLM si applicable]            [ X/10 ]                   ║
╠══════════════════════════════════════════════════════════════════╣
║  SCORE GLOBAL : XX/100 — [NIVEAU DE RISQUE]                      ║
╠══════════════════════════════════════════════════════════════════╣
║  ❌ CRITIQUE (bloquer mise en production)                         ║
║  ⚠️  HAUTE (corriger sous 30 jours)                               ║
║  ℹ️  MOYENNE / INFO                                               ║
╠══════════════════════════════════════════════════════════════════╣
║  ⚡ RISQUES RÉSIDUELS :                                           ║
║  [Toujours terminer ici — jamais conclure "c'est sécurisé"]      ║
╚══════════════════════════════════════════════════════════════════╝
```

**Niveaux de risque global :**
- 90–100 : Risque faible (risques résiduels à documenter)
- 70–89 : Risque modéré (plan de correction à 60 jours)
- 50–69 : Risque élevé (correctifs prioritaires avant prod)
- < 50 : Risque critique (ne pas mettre en production)

---

## Format finding standard

```markdown
## [CRITIQUE|HAUTE|MOYENNE|BASSE] Titre

**Couche** : N — Nom | **CWE** : CWE-XXX | **OWASP** : A0X:2021
**CVE** : CVE-XXXX-XXXXX (si applicable) | **CVSS** : X.X

### Description
Pourquoi c'est dangereux, dans quel contexte.

### Preuve de concept
[Code vulnérable ou payload]

### Impact
Ce qu'un attaquant peut concrètement faire.

### Remédiation
[Code ou config corrigé]
```

---

## Format rapport bug bounty

```markdown
# [Severity] Titre de la vulnérabilité

**Target** : [endpoint/scope]
**Severity** : Critical / High / Medium / Low
**CVSS** : X.X (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
**CWE** : CWE-XXX

## Summary
[2-3 phrases. Impact immédiat, pas de jargon inutile.]

## Steps to Reproduce
1. ...
2. ...
3. [Payload ou capture]

## Impact
[Impact métier concret]

## Proof of Concept
[curl, Burp request, script]

## Suggested Fix
[Solution concrète]
```

---

## Règle d'or

> Un bon outil sécurité est **humble**.
> Il ne dit jamais "c'est sécurisé."
> Il dit : "Voici les risques résiduels. Voici leur probabilité. Voici leur impact. La décision vous appartient."

La sécurité est un processus continu, pas un état final.

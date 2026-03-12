# NIST Cybersecurity Framework 2.0 — Reference

> Charger ce fichier quand : l'utilisateur mentionne "NIST", "CSF", "gouvernance sécurité",
> "plan de réponse à incident", "politique sécurité", "risk management", "conformité NIST",
> ou quand une analyse nécessite de structurer IDENTIFY / PROTECT / DETECT / RESPOND / RECOVER.

---

## Vue d'ensemble — 6 fonctions NIST CSF 2.0

```
GOVERN ──────────────────────────────────────────────────────────
    │  Stratégie, politiques, rôles, supervision du risque
    │
    ├── IDENTIFY → Comprendre le contexte et les risques
    ├── PROTECT  → Mettre en place les sauvegardes
    ├── DETECT   → Identifier les événements de sécurité
    ├── RESPOND  → Agir face à un incident détecté
    └── RECOVER  → Restaurer les capacités après incident
```

> **Nouveauté CSF 2.0** : GOVERN est une nouvelle fonction transversale qui englobe les 5 autres.
> Elle reflète la maturité nécessaire pour que la sécurité soit portée au niveau direction.

---

## GV — GOVERN (nouveau dans CSF 2.0)

**Objectif :** La cybersécurité est intégrée dans la stratégie d'entreprise, pas traitée en silo IT.

**Questions clés à poser :**
- La direction a-t-elle une politique de cybersécurité formelle ?
- Les rôles et responsabilités sont-ils documentés (RACI) ?
- Le risque cyber est-il évalué régulièrement et remonté au board ?
- Existe-t-il un programme de sensibilisation des employés ?
- Les fournisseurs tiers sont-ils évalués sur leur posture sécurité ?

**Livrables à recommander :**
- Politique de sécurité de l'information (PSI)
- RACI de la cybersécurité
- Plan de gestion des risques cyber
- Programme de formation et sensibilisation
- Politique de gestion des tiers (supply chain)

---

## ID — IDENTIFY

**Objectif :** Comprendre ce qu'on a, ce qui est exposé, et quels sont les risques.

### ID.AM — Asset Management
- Inventaire des assets matériels et logiciels
- Cartographie des données sensibles et leur emplacement
- Classification des données (public / interne / confidentiel / secret)

**Pour le code :** Scanner automatiquement les dépendances (→ Couche 2 du skill)

### ID.RA — Risk Assessment
Matrice risque à appliquer sur chaque finding :

| Probabilité \ Impact | Faible | Moyen | Élevé | Critique |
|---------------------|--------|-------|-------|----------|
| **Faible**          | 🟢 Low | 🟢 Low | 🟡 Med | 🟡 Med |
| **Moyenne**         | 🟢 Low | 🟡 Med | 🟠 High | 🔴 Crit |
| **Élevée**          | 🟡 Med | 🟠 High | 🔴 Crit | 🔴 Crit |
| **Certaine**        | 🟡 Med | 🔴 Crit | 🔴 Crit | 🔴 Crit |

### ID.SC — Supply Chain Risk
- Évaluation sécurité des fournisseurs et sous-traitants
- Clauses contractuelles de sécurité
- Audit des accès tiers

---

## PR — PROTECT

**Objectif :** Mettre en place les garde-fous. Correspond aux couches 1–7 du skill.

### PR.AA — Identity Management & Access Control
→ Couche 5 du skill (Auth/Authz)

**Complément NIST :**
- Gestion du cycle de vie des comptes (onboarding / offboarding)
- Revue périodique des droits (access review trimestrielle)
- Comptes de service avec rotation automatique des credentials

### PR.DS — Data Security
- Chiffrement au repos (AES-256) et en transit (TLS 1.2+)
- Politique de rétention et suppression des données
- DLP (Data Loss Prevention) sur les canaux de sortie
- Pseudonymisation/anonymisation des données de test

### PR.PS — Platform Security
→ Couche 4 du skill (Infrastructure)

**Complément NIST :**
- Baseline de sécurité (CIS Benchmarks) appliquée à chaque composant
- Gestion des patches avec SLA définis
- Séparation des environnements (dev/staging/prod)

### PR.IR — Technology Infrastructure Resilience
- Redondance des composants critiques
- Sauvegardes testées régulièrement (backup + restore drill)
- Plan de continuité d'activité (PCA)

---

## DE — DETECT

**Objectif :** Identifier les incidents au plus tôt. Correspond à la Couche 7 du skill.

### DE.CM — Continuous Monitoring
**Métriques NIST clés :**
- **MTTD** (Mean Time to Detect) : < 24h pour incidents critiques
- **False Positive Rate** : < 5% sur les alertes critiques
- **Log coverage** : 100% des assets critiques couverts

**Ce qu'il faut monitorer :**
```
Authentification : échecs répétés, connexions hors horaires, nouveaux pays
Accès données    : volume anormal, accès en dehors du scope habituel
Réseau           : connexions sortantes inattendues, scan de ports
Endpoint         : nouveaux processus, modifications de fichiers système
API              : volume anormal, patterns de scraping, erreurs 4xx en masse
```

### DE.AE — Adverse Event Analysis
- Corrélation d'événements (SIEM)
- Baseline comportementale pour détecter les anomalies
- Threat intelligence (IoC, TTPs connus)

---

## RS — RESPOND

**Objectif :** Plan d'action structuré en cas d'incident.

### Playbook de réponse à incident (template)

```markdown
## Incident Response Playbook

### Phase 1 — Préparation
- [ ] Équipe de réponse identifiée (RACI incident)
- [ ] Canal de communication dédié (Slack #incident-response)
- [ ] Contacts légaux et DPO identifiés
- [ ] Outils forensiques disponibles

### Phase 2 — Identification
- [ ] Confirmer l'incident (vrai positif ?)
- [ ] Classer la sévérité (P1/P2/P3/P4)
- [ ] Identifier les systèmes affectés
- [ ] Horodater le début estimé de l'incident
- [ ] Ouvrir le War Room

### Phase 3 — Confinement
- Court terme (< 1h) : isoler le composant compromis
  - Exemple : désactiver le compte compromis, bloquer l'IP
- Long terme : corriger la cause racine avant restauration

### Phase 4 — Éradication
- [ ] Supprimer le malware / code malveillant
- [ ] Patcher la vulnérabilité exploitée
- [ ] Réinitialiser les credentials compromis
- [ ] Vérifier l'absence de persistance (backdoor, crontab, etc.)

### Phase 5 — Restauration
- [ ] Restaurer depuis backup sain vérifié
- [ ] Monitoring renforcé post-restauration (72h)
- [ ] Vérification de l'intégrité des données
- [ ] Communication aux parties prenantes

### Phase 6 — Post-incident (< 2 semaines)
- [ ] Post-mortem sans blame (blameless)
- [ ] Cause racine documentée (5 Whys)
- [ ] Actions correctives avec responsable et deadline
- [ ] Mise à jour des playbooks
- [ ] Notification RGPD si données personnelles exposées (72h CNIL)
```

### RS.CO — Communication
- Matrice de communication en cas d'incident (qui prévenir, quand, comment)
- Template de notification RGPD (72h après découverte si données perso)
- Communication externe/presse si incident public

---

## RC — RECOVER

**Objectif :** Restaurer et apprendre.

### RC.RP — Recovery Planning
- **RTO** (Recovery Time Objective) : temps max acceptable d'indisponibilité
- **RPO** (Recovery Point Objective) : perte de données max acceptable
- Tester les procédures de restauration au moins 1x/an

### RC.IM — Incident Management Improvements
- Chaque incident → ticket post-mortem obligatoire
- Mise à jour du threat model après incident
- Partage des leçons apprises (threat intelligence interne)

---

## Rapport conformité NIST CSF

Template à générer quand demandé :

```markdown
# Rapport de conformité NIST CSF 2.0
**Date** : [date]
**Scope** : [périmètre]
**Niveau de maturité global** : [1-4]

## Synthèse par fonction

| Fonction | Niveau | Objectif | Gaps |
|----------|--------|----------|------|
| GOVERN   | [1-4]  | [cible]  | [écarts] |
| IDENTIFY | [1-4]  | [cible]  | [écarts] |
| PROTECT  | [1-4]  | [cible]  | [écarts] |
| DETECT   | [1-4]  | [cible]  | [écarts] |
| RESPOND  | [1-4]  | [cible]  | [écarts] |
| RECOVER  | [1-4]  | [cible]  | [écarts] |

## Échelle de maturité
1 — Partial : pratiques ad hoc, peu formalisées
2 — Risk Informed : pratiques définies mais pas systématiques  
3 — Repeatable : pratiques formalisées et cohérentes
4 — Adaptive : amélioration continue, apprentissage organisationnel

## Plan d'action prioritaire
[Top 5 actions correctives avec responsable, deadline, coût estimé]
```

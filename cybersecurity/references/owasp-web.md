# OWASP Web Top 10 — 2021 Reference

> Charger ce fichier quand : audit web, code review, génération API/backend, pentest web, bug bounty web app.

## Mapping complet — 10 risques critiques

---

### A01:2021 — Broken Access Control (↑ #1)

**Ce que le skill doit chercher :**
- Accès à ressources d'autres utilisateurs sans vérification propriétaire (IDOR)
- Élévation de privilège horizontal et vertical
- Manipulation de métadonnées (JWT, cookie, champ caché `role=admin`)
- CORS mal configuré autorisant des origines non fiables
- Accès à des routes admin sans contrôle `@PreAuthorize`
- `allowedRoles` vérifié uniquement côté frontend

**Payloads de test :**
```bash
# IDOR basique
GET /api/orders/1337 HTTP/1.1
Authorization: Bearer <token_user_42>

# Privilege escalation via paramètre
POST /api/profile/update
{"userId": 1, "role": "ADMIN"}

# JWT role tampering (si HS256 avec secret faible)
# Modifier payload base64 : "role":"user" → "role":"admin"
```

**Fix Spring Boot :**
```java
@GetMapping("/orders/{id}")
public Order getOrder(@PathVariable Long id, Principal principal) {
    Order order = orderRepo.findById(id).orElseThrow();
    if (!order.getUserId().equals(getCurrentUserId(principal))) {
        throw new AccessDeniedException("Accès refusé");
    }
    return order;
}
```

**CWE** : CWE-200, CWE-284, CWE-639 | **CVSS typique** : 6.5–9.8

---

### A02:2021 — Cryptographic Failures (↑ #2)

**Ce que le skill doit chercher :**
- Données sensibles transmises en HTTP clair
- Algorithmes obsolètes : MD5, SHA1, DES, RC4, ECB mode
- Clés hardcodées ou entropie insuffisante
- JWT HS256 avec secret faible (bruteforçable)
- Données PII en clair en base (non chiffrées au repos)
- Certificats expirés ou auto-signés en production
- `TLS 1.0` / `TLS 1.1` encore activés

**Fix :**
```java
// ✅ Hash password
PasswordEncoder encoder = new BCryptPasswordEncoder(12);

// ✅ Chiffrement AES-GCM (pas ECB)
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");

// ✅ TLS config Spring Boot
server.ssl.enabled=true
server.ssl.protocol=TLS
server.ssl.enabled-protocols=TLSv1.3,TLSv1.2
```

**CWE** : CWE-261, CWE-326, CWE-327, CWE-328

---

### A03:2021 — Injection (↓ #3)

**Vecteurs à couvrir :**

| Type | Signal | Payload test |
|------|---------|-------------|
| SQLi | Concaténation string dans query | `' OR '1'='1` |
| NoSQLi | `$where`, opérateurs Mongo non filtrés | `{"$gt": ""}` |
| OS Command | `Runtime.exec(userInput)` | `; cat /etc/passwd` |
| LDAP | Filtre construit dynamiquement | `*)(uid=*))(|(uid=*` |
| SSTI | Template + variable utilisateur | `{{7*7}}`, `${7*7}` |
| XXE | Parser XML sans sécurisation | `<!ENTITY xxe SYSTEM "file:///etc/passwd">` |
| XSS | `innerHTML`, `dangerouslySetInnerHTML` | `<img src=x onerror=alert(1)>` |

**Fix universel :** Validation entrée → Paramétrage requête → Encodage sortie

**CWE** : CWE-74, CWE-77, CWE-89, CWE-564, CWE-643

---

### A04:2021 — Insecure Design

**Ce que le skill doit chercher :**
- Absence de modélisation des menaces (threat model)
- Logique métier sans cas d'abus documentés
- Pas de rate limiting dès la conception
- Pas de principe de moindre privilège dans l'architecture
- Flux de récupération de mot de passe contournables
- Questions secrètes comme unique facteur de récupération

**Action :** Déclencher le threat model STRIDE automatiquement si le design semble non réfléchi.

---

### A05:2021 — Security Misconfiguration

**Checklist :**
- [ ] Headers de sécurité HTTP absents (CSP, HSTS, X-Frame-Options)
- [ ] Stack trace exposé en production (`server.error.include-stacktrace=always`)
- [ ] Ports/services inutiles ouverts
- [ ] Comptes par défaut non changés
- [ ] Directory listing activé
- [ ] Mode debug activé en prod
- [ ] S3 bucket public par erreur
- [ ] Swagger UI exposé en production sans auth

**Fix Spring Boot :**
```properties
# Désactiver stack traces en prod
server.error.include-stacktrace=never
server.error.include-message=never
spring.jpa.show-sql=false
```

**CWE** : CWE-16, CWE-2

---

### A06:2021 — Vulnerable and Outdated Components

→ **Couche 2 du skill (Dépendances)** — référence principale.

Points supplémentaires :
- Surveiller le GitHub Security Advisory feed
- Intégrer OWASP Dependency-Check ou Snyk en CI/CD
- Politique de mise à jour : CRITICAL → patch sous 24h, HIGH → 7 jours, MEDIUM → 30 jours

---

### A07:2021 — Identification and Authentication Failures

→ **Couche 5 du skill (Auth/Authz)** — référence principale.

Points supplémentaires :
- Énumération d'utilisateurs via messages d'erreur différenciés
- `Timing attack` sur comparaison de tokens (`==` vs `MessageDigest.isEqual()`)
- Absence de `account lockout` progressif

---

### A08:2021 — Software and Data Integrity Failures

**Ce que le skill doit chercher :**
- CI/CD pipeline sans vérification d'intégrité des artifacts
- Désérialisation d'objets non fiables sans signature
- Mise à jour automatique sans vérification cryptographique
- Dépendances chargées depuis CDN public sans `integrity` hash (SRI)

**Fix HTML :**
```html
<!-- Subresource Integrity -->
<script src="https://cdn.example.com/lib.js"
        integrity="sha384-[hash]"
        crossorigin="anonymous"></script>
```

**CWE** : CWE-494, CWE-502, CWE-829

---

### A09:2021 — Security Logging and Monitoring Failures

→ **Couche 7 du skill (Observabilité)** — référence principale.

Ajout : OWASP recommande un **MTTD** (Mean Time To Detect) < 24h pour les incidents critiques.

---

### A10:2021 — Server-Side Request Forgery (SSRF)

**Ce que le skill doit chercher :**
- Endpoints qui récupèrent une URL fournie par l'utilisateur
- Accès aux métadonnées cloud (`http://169.254.169.254/`)
- Bypass de firewall interne via le serveur
- Webhooks sans validation de l'URL cible

**Payload de test :**
```bash
# Accès métadonnées AWS
POST /api/fetch-url
{"url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/"}

# Scan réseau interne
{"url": "http://192.168.1.1/admin"}

# Bypass via redirection
{"url": "http://attacker.com/redirect?to=http://internal-service"}
```

**Fix :**
```java
// Whitelist des domaines autorisés
List<String> allowedHosts = List.of("api.trusted.com", "cdn.myapp.com");
URL url = new URL(userInput);
if (!allowedHosts.contains(url.getHost())) {
    throw new SecurityException("SSRF attempt blocked");
}
```

**CWE** : CWE-918 | **CVSS typique** : 7.5–9.8

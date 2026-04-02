# JWT — Authentification

JSON Web Token (JWT) est un standard ouvert (RFC 7519) pour transmettre des informations de façon sécurisée entre deux parties sous forme de token signé. Il est largement utilisé pour l'authentification dans les APIs REST et les SPAs.

---

## Anatomie d'un JWT

Un JWT est une chaîne en trois parties séparées par des points : `header.payload.signature`

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.eyJzdWIiOiIxMjM0NSIsImVtYWlsIjoiYWxpY2VAZXhlbXBsZS5jb20iLCJyb2xlIjoidXNlciIsImlhdCI6MTcwMDAwMDAwMCwiZXhwIjoxNzAwMDAwOTAwfQ
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Chaque partie est encodée en **base64url** (pas chiffrée — lisible par quiconque).

### Header

```json
{
  "alg": "HS256",   // algorithme de signature
  "typ": "JWT"
}
```

### Payload (claims)

```json
{
  "sub": "12345",                    // subject — identifiant de l'utilisateur
  "email": "alice@exemple.com",
  "role": "user",
  "iss": "mon-app",                  // issuer — émetteur du token
  "aud": "mon-api",                  // audience — destinataire prévu
  "iat": 1700000000,                 // issued at — date d'émission (Unix timestamp)
  "exp": 1700000900                  // expiration — date d'expiration (Unix timestamp)
}
```

> ⚠️ Le payload est **lisible par tous** — ne jamais y stocker de mot de passe, numéro de carte, ou données sensibles.

### Signature

```
HMACSHA256(
  base64url(header) + "." + base64url(payload),
  SECRET
)
```

La signature garantit que le token n'a pas été altéré. Elle ne chiffre pas le contenu.

---

## Générer un token (login)

```bash
npm install jsonwebtoken
```

```js
const jwt = require('jsonwebtoken');

// ❌ Vulnérable : secret hardcodé, pas d'expiration
const token = jwt.sign({ userId: user.id }, 'secret');

// ✅ Corrigé : secret depuis l'env, durées courtes, claims complets
const ACCESS_TOKEN_SECRET = process.env.ACCESS_TOKEN_SECRET;
const REFRESH_TOKEN_SECRET = process.env.REFRESH_TOKEN_SECRET;

function genererTokens(user) {
  const accessToken = jwt.sign(
    { userId: user.id, email: user.email, role: user.role },
    ACCESS_TOKEN_SECRET,
    { expiresIn: '15m', issuer: 'mon-app', audience: 'mon-api' }
  );

  const refreshToken = jwt.sign(
    { userId: user.id },
    REFRESH_TOKEN_SECRET,
    { expiresIn: '7d', issuer: 'mon-app' }
  );

  return { accessToken, refreshToken };
}
```

```js
// Endpoint de login
app.post('/api/login', async (req, res) => {
  const user = await verifierIdentifiants(req.body.email, req.body.password);
  if (!user) return res.status(401).json({ erreur: 'Identifiants invalides' });

  const { accessToken, refreshToken } = genererTokens(user);

  // Stocker le refresh token en base (voir section Révocation)
  await db.sauvegarderRefreshToken(user.id, refreshToken);

  res.json({ accessToken, refreshToken, tokenType: 'Bearer', expiresIn: '15m' });
});
```

---

## Vérifier un token (middleware)

```js
// middleware/authentifie.js
const jwt = require('jsonwebtoken');

function authentifie(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader?.split(' ')[1]; // "Bearer <token>"

  if (!token) {
    return res.status(401).json({ erreur: 'Token manquant' });
  }

  jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, { audience: 'mon-api' }, (err, decoded) => {
    if (err) {
      if (err instanceof jwt.TokenExpiredError) {
        return res.status(401).json({ erreur: 'Token expiré', code: 'TOKEN_EXPIRED' });
      }
      if (err instanceof jwt.JsonWebTokenError) {
        return res.status(403).json({ erreur: 'Token invalide', code: 'TOKEN_INVALID' });
      }
      return res.status(500).json({ erreur: 'Erreur de vérification' });
    }

    req.user = decoded; // { userId, email, role, iat, exp, ... }
    next();
  });
}

module.exports = authentifie;
```

```js
// Utilisation sur les routes protégées
app.get('/api/profil', authentifie, (req, res) => {
  res.json({ userId: req.user.userId, email: req.user.email });
});
```

---

## Refresh token

L'access token a une courte durée de vie (15 min). Le refresh token (7 jours) permet d'en obtenir un nouveau sans redemander le mot de passe.

```
Client                          Serveur
  │── POST /api/login ─────────────→│  accessToken (15m) + refreshToken (7j)
  │                                 │
  │── GET /api/données (Bearer) ───→│  ✅ ok tant que accessToken valide
  │                                 │
  │   [accessToken expiré]          │
  │── GET /api/données (Bearer) ───→│  401 TOKEN_EXPIRED
  │                                 │
  │── POST /api/refresh ────────────→│  nouveau accessToken
  │        { refreshToken }         │
  │←── { accessToken } ─────────────│
  │                                 │
  │── GET /api/données (Bearer) ───→│  ✅ ok avec le nouveau token
```

```js
// Endpoint de refresh
app.post('/api/refresh', async (req, res) => {
  const { refreshToken } = req.body;

  if (!refreshToken) {
    return res.status(401).json({ erreur: 'Refresh token manquant' });
  }

  // Vérifier que le token est en base (non révoqué)
  const tokenEnBase = await db.trouverRefreshToken(refreshToken);
  if (!tokenEnBase) {
    return res.status(403).json({ erreur: 'Refresh token invalide ou révoqué' });
  }

  jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET, async (err, decoded) => {
    if (err) {
      await db.supprimerRefreshToken(refreshToken); // nettoyer si expiré
      return res.status(403).json({ erreur: 'Refresh token expiré' });
    }

    // Rotation : invalider l'ancien, émettre un nouveau
    await db.supprimerRefreshToken(refreshToken);
    const user = await db.trouverUtilisateur(decoded.userId);
    const { accessToken, refreshToken: nouveauRefresh } = genererTokens(user);
    await db.sauvegarderRefreshToken(user.id, nouveauRefresh);

    res.json({ accessToken, refreshToken: nouveauRefresh });
  });
});

// Déconnexion : invalider le refresh token
app.post('/api/logout', authentifie, async (req, res) => {
  const { refreshToken } = req.body;
  if (refreshToken) await db.supprimerRefreshToken(refreshToken);
  res.json({ message: 'Déconnecté' });
});
```

---

## Stockage côté client

### Comparatif

| | `localStorage` | Cookie `httpOnly` |
|---|---|---|
| **Accessible par JS** | Oui | Non |
| **Risque XSS** | Élevé — script malveillant peut lire le token | Faible — inaccessible depuis JS |
| **Risque CSRF** | Faible | Modéré — mitigé par `SameSite=Strict` |
| **Persistance** | Jusqu'à effacement | Configurable (session ou date d'exp.) |
| **Recommandé pour** | Prototypes / apps sans données sensibles | Applications en production |

### Option A — Authorization header (localStorage ou mémoire)

```js
// ❌ localStorage : vulnérable au XSS
localStorage.setItem('accessToken', token);
const token = localStorage.getItem('accessToken');

// ✅ Mémoire JS (variable de module) : non persistant mais sécurisé contre XSS
let accessToken = null;

async function login(email, password) {
  const res = await fetch('/api/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  });
  const data = await res.json();
  accessToken = data.accessToken; // stocké en mémoire uniquement
}

// Requête authentifiée
async function fetchProfil() {
  const res = await fetch('/api/profil', {
    headers: { 'Authorization': `Bearer ${accessToken}` },
  });
  return res.json();
}
```

### Option B — Cookie httpOnly (recommandé en production)

```js
// Serveur : envoyer le token dans un cookie httpOnly
res.cookie('accessToken', accessToken, {
  httpOnly: true,     // inaccessible depuis JS
  secure: true,       // HTTPS uniquement
  sameSite: 'Strict', // protection CSRF
  maxAge: 15 * 60 * 1000, // 15 minutes
});

// Le navigateur envoie automatiquement le cookie — pas de header à gérer côté client
const res = await fetch('/api/profil', { credentials: 'include' });
```

```js
// Middleware côté serveur : lire depuis le cookie plutôt que le header
function authentifie(req, res, next) {
  const token = req.cookies?.accessToken || req.headers['authorization']?.split(' ')[1];
  // ... vérification identique
}
```

---

## Révocation et sécurité

### Pourquoi les JWT sont difficiles à révoquer

Par nature, un JWT est **auto-suffisant** : le serveur n'a pas besoin de consulter une base pour le valider. La contrepartie : il est valide jusqu'à expiration, même après déconnexion.

### Stratégies

```
1. Access tokens à courte durée (15 min)
   → La fenêtre de risque est limitée même si un token est volé

2. Rotation des refresh tokens
   → À chaque refresh, l'ancien token est invalidé
   → Si un token volé est réutilisé, la rotation détecte la collision

3. Blacklist (pour révocation immédiate)
   → Stocker les JTI (JWT ID) révoqués dans Redis avec TTL = durée d'expiration
```

```js
// Ajouter un identifiant unique au token
const accessToken = jwt.sign(
  { userId: user.id, jti: crypto.randomUUID() },
  process.env.ACCESS_TOKEN_SECRET,
  { expiresIn: '15m' }
);

// Dans le middleware : vérifier que le jti n'est pas blacklisté
const estRevoque = await redis.get(`blacklist:${decoded.jti}`);
if (estRevoque) return res.status(401).json({ erreur: 'Token révoqué' });

// À la déconnexion : blacklister le token jusqu'à son expiration
const ttl = decoded.exp - Math.floor(Date.now() / 1000);
await redis.set(`blacklist:${decoded.jti}`, '1', { EX: ttl });
```

### Algorithmes : HS256 vs RS256

| | HS256 (symétrique) | RS256 (asymétrique) |
|---|---|---|
| **Clé de signature** | Secret partagé | Clé privée |
| **Clé de vérification** | Même secret | Clé publique |
| **Usage** | Monolithe, microservices de confiance | Microservices, fournisseur d'identité externe |
| **Avantage** | Simple, rapide | La clé publique peut être distribuée sans risque |

```js
// RS256 : signer avec la clé privée, vérifier avec la publique
const fs = require('fs');
const privateKey = fs.readFileSync('private.key');
const publicKey = fs.readFileSync('public.pem');

const token = jwt.sign({ userId: 123 }, privateKey, { algorithm: 'RS256' });
const decoded = jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

---

## Checklist

| # | Vérification |
|---|---|
| 1 | Secrets JWT dans les variables d'environnement (jamais dans le code) |
| 2 | Access token : durée ≤ 15 minutes |
| 3 | Refresh token : stocké en base, invalidé à la déconnexion |
| 4 | Rotation des refresh tokens activée |
| 5 | Payload ne contient pas de données sensibles |
| 6 | Token stocké en mémoire JS ou cookie `httpOnly` (pas `localStorage`) |
| 7 | Cookie avec `Secure`, `httpOnly`, `SameSite=Strict` |
| 8 | Middleware vérifie `aud` et `iss` |
| 9 | Erreurs JWT : messages génériques côté client, détails en log serveur |
| 10 | Algorithme explicitement spécifié (`algorithms: ['HS256']`) |

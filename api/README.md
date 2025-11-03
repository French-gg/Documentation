# API French.gg - Documentation

## 🔗 Informations Générales

L'API French.gg est développée en TypeScript avec Fastify et utilise Prisma ORM pour la gestion de la base de données MySQL.

- **Base URL** : `https://api.french.gg`
- **Version actuelle** : v1
- **Format** : JSON
- **Authentification** : JWT + Cookies sécurisés

## 🏗️ Architecture

### Stack Technique
- **Framework** : Fastify (Node.js)
- **Language** : TypeScript
- **ORM** : Prisma
- **Base de données** : MySQL/MariaDB
- **Cache** : Node-Cache
- **Logs** : Pino avec pino-pretty

### Structure du Projet
```
_api/
├── prisma/
│   └── schema.prisma        # Schéma de base de données
├── src/
│   ├── index.ts            # Point d'entrée principal
│   ├── handler/            # Gestionnaires de requêtes
│   ├── preHandler/         # Middleware de pré-traitement
│   ├── routes/             # Définition des routes
│   │   ├── v1/
│   │   │   ├── admin/      # Routes d'administration
│   │   │   ├── auth/       # Authentification
│   │   │   ├── discord/    # Intégration Discord
│   │   │   ├── health/     # Santé du service
│   │   │   ├── servers/    # Gestion des serveurs
│   │   │   ├── users/      # Gestion des utilisateurs
│   │   │   └── utils/      # Utilitaires
│   │   └── v2/            # Version 2 (développement)
│   ├── structure/          # Classes de base
│   ├── type/              # Types TypeScript
│   └── utils/             # Fonctions utilitaires
└── package.json
```

## 🔐 Authentification

### Flux OAuth Discord
1. Redirection vers Discord OAuth
2. Réception du code d'autorisation
3. Échange contre un token d'accès
4. Création de session utilisateur
5. Génération du JWT
6. Stockage sécurisé en cookie

### Gestion des Sessions
```typescript
interface FggSession {
  uuid: string;           // Identifiant unique de session
  snowflake: bigint;     // Snowflake de session
  user_id: bigint;       // ID utilisateur French.gg
  ds_user_id: string;    // ID utilisateur Discord
  ds_token: string;      // Token d'accès Discord
  ds_refresh: string;    // Token de refresh Discord
  ds_expires_at: Date;   // Expiration du token Discord
  created_at: Date;      // Date de création
  expires_at: Date;      // Date d'expiration
}
```

## 📊 Endpoints Principaux

### Health Check
```http
GET /v1/health
```
**Réponse :**
```json
{
  "status": "OK",
  "uptime": 123456,
  "timestamp": "2024-11-03T10:30:00Z"
}
```

### Authentification
```http
POST /v1/auth/discord
Content-Type: application/json
{
  "code": "discord_oauth_code",
  "redirect_uri": "https://french.gg/callback"
}
```

### Serveurs
```http
GET /v1/servers
GET /v1/servers/:id
POST /v1/servers (authentification requise)
PUT /v1/servers/:id (authentification requise)
DELETE /v1/servers/:id (authentification requise)
```

### Utilisateurs
```http
GET /v1/users/profile/:id
GET /v1/users/me (authentification requise)
PUT /v1/users/me (authentification requise)
```

## 🛡️ Sécurité

### CORS Configuration
```typescript
origin: [
  'https://french.gg',
  'https://auth.french.gg',
  'https://admin.french.gg',
  'https://canary.french.gg',
  'https://frgg.me'
]
```

### Cookies Sécurisés
- `SameSite: 'none'`
- `Secure: true`
- `HttpOnly: true`
- Expiration automatique

### Validation des Données
- Schémas Fastify JSON Schema
- Validation des paramètres
- Sanitisation des entrées
- Protection contre l'injection SQL (Prisma)

## 🗄️ Modèles de Données

### Utilisateurs
```prisma
model fgg_user {
  id      BigInt  @id @unique
  email   String  @unique @db.VarChar(255)
  
  fgg_session         fgg_session[]
  fgg_user_discord    fgg_user_discord[]
}
```

### Sessions
```prisma
model fgg_session {
  uuid           String   @id @unique @db.VarChar(255)
  snowflake      BigInt   @default(0) @db.UnsignedBigInt
  user_id        BigInt
  ds_user_id     String
  ds_token       String   @db.VarChar(255)
  ds_refresh     String   @db.VarChar(255)
  ds_expires_at  DateTime
  created_at     DateTime @default(now())
  expires_at     DateTime
}
```

## 🚀 Déploiement

### Variables d'Environnement
```bash
DATABASE_URL="mysql://user:password@localhost:3306/frenchgg"
DISCORD_CLIENT_ID="your_discord_client_id"
DISCORD_CLIENT_SECRET="your_discord_client_secret"
JWT_SECRET="your_jwt_secret"
NODE_ENV="production"
```

### Scripts de Démarrage
```bash
# Développement
npm run dev

# Production
npm run build
npm start
```

## 📈 Monitoring

### Logs Structurés
- Niveau de log configurable
- Format JSON en production
- Rotation automatique des fichiers
- Intégration avec les outils de monitoring

### Métriques
- Temps de réponse par endpoint
- Taux d'erreur
- Utilisation mémoire
- Connexions base de données

## 🔄 Gestion des Erreurs

### Codes de Statut
- `200` : Succès
- `400` : Requête invalide
- `401` : Non authentifié
- `403` : Non autorisé
- `404` : Ressource non trouvée
- `429` : Trop de requêtes
- `500` : Erreur serveur

### Format des Erreurs
```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Description de l'erreur",
    "details": "Informations supplémentaires"
  }
}
```

---

**Liens utiles :**
- [Schéma de base de données](./database.md)
- [Guide d'authentification](./auth.md)
- [Tests API](./tests.md)
# Plans Moyen Terme — Prospection bureau-mq.com

## 1. Dashboard Stats (Priorité 1)

### Ce qu'on veut
Un tableau de bord visuel qui remplace "des réponses ?" par un coup d'œil.

### Stack
Go + Unpoly + SQLite (stack existante). Rien de plus. Le dashboard lit la DB en lecture seule.

### Pages
```
/prospection/dashboard?token=xxx
├── KPI cards (envoyés, réponses, bounces, relances)
├── Graphique envois/jour (derniers 30j)
├── Tableau derniers envoyés (avec statut réponse)
├── Tableau prochaines relances
└── Répartition par secteur (pie chart CSS)
```

### Fichiers
```
internal/
├── adapters/
│   └── httpserver/
│       └── handlers/
│           └── dashboard.go    # Nouveau handler
web/templates/pages/dashboard.gohtml
web/static/css/dashboard.css
```

### Données exposées
```sql
-- KPIs
SELECT COUNT(*) FROM prospects WHERE statut = 'contacte';           -- envoyés
SELECT COUNT(*) FROM prospects WHERE statut IN ('contacte','repondu');-- réponses
SELECT COUNT(*) FROM prospects WHERE email_status = 'bounce';        -- bounces

-- Envois par jour (7 derniers jours)
SELECT DATE(date_envoi), COUNT(*) FROM prospects 
WHERE date_envoi > DATE('now','-7 days') GROUP BY 1;

-- Relances à venir
SELECT nom, email, date_envoi FROM prospects 
WHERE statut = 'contacte' AND date_envoi <= DATE('now','-3 days')
AND followup_sent = 0;

-- Par secteur
SELECT secteur, COUNT(*) FROM prospects WHERE statut = 'contacte' GROUP BY 1;
```

### Design
Fond sombre (comme la démo vidéo pipeline), cartes avec chiffres gros, couleurs vert/jaune/rouge.

### Sécurité
Un token durci dans l'URL (`?token=sha256hash`). Pas de login, c'est un dashboard interne.

### Effort : 1 jour

---

## 2. WhatsApp Business (Priorité 2)

### Approche recommandée : Baileys (open-source)

Baileys est une lib Node.js qui émule WhatsApp Web. Gratuit, pas d'API Meta à approuver.

### Architecture
```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  Go pipeline │────▶│  Redis queue  │────▶│  Baileys    │
│  (génère msg)│     │  (messages)   │     │  (envoie)   │
└─────────────┘     └──────────────┘     └─────────────┘
                                              │
                                              ▼
                                         WhatsApp
```

### Fichiers
```
whatsapp-bridge/
├── package.json
├── bot.js              # Baileys main
├── auth/               # Session WhatsApp (générée au 1er scan QR)
├── queue.ts            # Lit Redis, envoie messages
└── .env
```

### Flow
1. Go pipeline : prospect enrichi avec téléphone → écrit message dans DB + Redis
2. Baileys bridge : lit la queue, envoie le message WhatsApp
3. Status callback : marque "envoyé" / "lu" dans la DB

### Messages template
```
Bonjour {{.Prenom}} 👋

Je suis Alexandre, développeur en Martinique.
J'aide les {{.Secteur}} à automatiser leur {{.Probleme}}.

Ça vous dit qu'on en parle 10 minutes cette semaine ?
```

### Coût
- Baileys : gratuit
- Redis : gratuit (ou SQLite queue)
- Téléphone : ton numéro pro existant
- **Total : $0/mo**

### Risques
- WhatsApp peut bannir les comptes qui envoient trop de messages (max ~50/jour en froid)
- Baileys casse parfois quand WhatsApp change son protocole

### Effort : 2 jours

---

## 3. LinkedIn Outreach (Priorité 3)

### Approche : semi-automatique

Le scraping LinkedIn 100% automatique est casse-gueule (bans IP, légal). On fait du **semi-auto** : un script qui prépare tout, un humain qui valide et envoie.

### Flow
```
┌─────────────────┐
│ Go : scrape     │  Recherche "Martinique" + filtre secteur
│ LinkedIn Search │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Go : enrich     │  Croise avec DB prospects existante
│ Match existing  │  Ajoute les nouveaux décideurs
└────────┬────────┘
         ▼
┌─────────────────┐
│ CSV output      │  Fichier prêt à être importé
│ Pour import     │  dans Sales Navigator / manual
└─────────────────┘
```

### Pourquoi semi-auto ?
- LinkedIn bloque le scraping automatisé depuis 2024 (loi antitrust + technique)
- Mieux vaut exporter une liste propre et la passer en manuel ou via un outil compliant
- Le CRM Apollo.io fait ça nativement ($49/mo)

### Alternative full-auto : Apollo.io API
- 275M contacts B2B
- Filtres : Martinique + secteur + poste
- Export automatique → ta DB SQLite
- Coût : $49/mo (plan Basic)
- **Effort** : 2h d'intégration API

### Fichiers
```
internal/
├── adapters/
│   └── linkedin/
│       ├── scrape.go      # Recherche + parsing
│       └── enrich.go      # Match avec DB
└── cmd/
    └── linkedin-sync/      # CLI : go run . linkedin-sync
```

### Effort : 1-2 jours (semi-auto) ou 2h (Apollo API)

---

## 4. Sauvegardes DB automatiques

### Script de backup
```bash
#!/bin/bash
# ~/.hermes/scripts/backup_prospection_db.sh
BACKUP_DIR=~/prospection/backups
mkdir -p "$BACKUP_DIR"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
cp ~/prospection/data/prospection.db "$BACKUP_DIR/prospection_$TIMESTAMP.db"
# Garder les 30 derniers backups
ls -t "$BACKUP_DIR"/prospection_*.db | tail -n +31 | xargs -r rm
echo "Backup: prospection_$TIMESTAMP.db ($(du -h "$BACKUP_DIR/prospection_$TIMESTAMP.db" | cut -f1))"
```

### Cron : toutes les heures
```
0 * * * * ~/.hermes/scripts/backup_prospection_db.sh
```

### Restauration
```bash
cp ~/prospection/backups/prospection_20260714_120000.db ~/prospection/data/prospection.db
```

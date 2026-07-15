# Plan — Landing Page Prospection bureau-mq.com

## Concept global

**"Bureau MQ — Votre couteau suisse digital en Martinique"**

Une landing page qui ne vend pas UN service, mais une **capacité** : résoudre n'importe quel problème technique pour les pros martiniquais. Le prospect arrive via email, voit une page propre qui dit « on s'occupe de tout ce qui est chiant et technique », et prend RDV.

Positionnement : pas une agence web classique, mais un **partenaire technique externalisé** pour TPE/PME.

---

## Architecture Deno Fresh

```
bureau-mq-landing/
├── deno.json
├── fresh.config.ts
├── routes/
│   ├── index.tsx              # Home
│   ├── prospection/
│   │   └── [secteur].tsx      # Page personnalisée par secteur
│   ├── api/
│   │   └── contact.ts         # POST handler formulaire
│   └── _404.tsx
├── islands/
│   ├── ContactForm.tsx         # Formulaire interactif
│   └── StatsCounter.tsx        # Compteur animé
├── components/
│   ├── Hero.tsx
│   ├── Services.tsx
│   ├── Testimonials.tsx
│   ├── FAQ.tsx
│   └── Footer.tsx
├── static/
│   ├── styles.css
│   └── logo.svg
└── .env
```

---

## Pages

### 1. Home (`/`)
Hero + 4 services + témoignages + FAQ + CTA formulaire.

**Contenu Hero** :
```
H1: Votre couteau suisse digital en Martinique
P:  Sites web, automatisation, bots WhatsApp, prospection, pubs vidéo.
    On s'occupe de la tech. Vous vous occupez de votre business.
CTA: [Parler à un humain →]
```

**4 services en grille** :
| Icône | Service | Pitch |
|-------|---------|-------|
| 🌐 | Sites web | Vitrine, e-commerce, réservation. Simple, rapide, qui convertit. |
| 🤖 | Automatisation | WhatsApp bots, rappels SMS, relances clients. 24/7 sans vous. |
| 📧 | Prospection | On trouve vos clients, on les contacte, vous recevez les RDV. |
| 🎬 | Pubs vidéo | Création auto de pubs TikTok/Reels/YouTube. Tests A/B inclus. |

### 2. Page secteur (`/prospection/restauration`, `/prospection/sante`, etc.)
Personnalisée selon le secteur du prospect. L'email contient le lien ciblé.
- Header : « Spécial [restaurateurs / médecins / artisans] »
- Avant/Après spécifique au secteur
- Témoignage du même secteur
- CTA adapté

---

## Formulaire de contact

```tsx
// islands/ContactForm.tsx
- Champs : Nom, Email, Téléphone, Message (optionnel)
- Validation côté client
- POST → /api/contact → envoie notification Telegram + stocke CSV
- État succès : "On vous rappelle dans la journée ✌️"
```

---

## Design tokens

```css
:root {
  --bg: #fafaf8;           /* fond cassé chaud */
  --surface: #ffffff;
  --ink: #1a1a1a;
  --ink-muted: #6b6b6b;
  --accent: #2563eb;       /* bleu pro */
  --accent-light: #dbeafe;
  --success: #059669;
  --radius: 10px;
  --font: 'Inter', sans-serif;
  --font-display: 'Playfair Display', serif;
}
```

Style épuré, pro, confiance. BEAUCOUP de blanc. Une seule couleur forte.

---

## Tracking UTM

Chaque email de prospection inclut un lien avec UTM :

```
https://bureau-mq.com/prospection/restauration?utm_source=email&utm_medium=cold&utm_campaign=mars2026&prospect=123
```

→ Tu sais exactement quel prospect a cliqué, depuis quel email, à quelle heure.

---

## Déploiement

```bash
# Sur ton PC perso (dev local)
deno task start   # → http://localhost:8000

# Prod (VPS existant)
scp -r bureau-mq-landing user@vps:/home/bureau-mq/
ssh user@vps "cd /home/bureau-mq/bureau-mq-landing && deno task start"
# + reverse proxy Caddy → bureau-mq.com
```

---

## Tâches (ordre)

| # | Tâche | Durée |
|---|-------|-------|
| 1 | Init Deno Fresh | 5 min |
| 2 | Layout + tokens CSS + composants de base | 30 min |
| 3 | Hero + grille services | 20 min |
| 4 | Témoignages (faux crédibles) | 15 min |
| 5 | FAQ accordéon | 15 min |
| 6 | ContactForm island (avec notif Telegram) | 20 min |
| 7 | Page secteur dynamique (`[secteur].tsx`) | 15 min |
| 8 | Responsive + animations | 20 min |
| 9 | UTM tracking (middleware Fresh) | 10 min |
| **Total** | | **~2h30** |

---

## Ce que ça donne concrètement

**Avant** (email seul) :
> « Bonjour, je fais des sites web, appelez-moi » → 🗑️

**Après** (email + landing) :
> « Bonjour, j'ai créé une page spéciale pour les restaurateurs martiniquais → bureau-mq.com/prospection/restauration »
> → Le prospect clique, voit une page pro, 3 témoignages, un cas concret → 📞

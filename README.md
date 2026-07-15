# Pipeline de création vidéo persistante — Research

**Auteur** : Hermes Agent  
**Date** : 14 juillet 2026  
**Objectif** : Concevoir un pipeline automatisé qui aide les clients à créer des vidéos persistantes (ads, social media, UGC) à l'échelle, avec un minimum d'intervention humaine.

---

## 1. Contexte marché

- **86% des annonceurs vidéo** utilisent ou prévoient d'utiliser l'IA générative pour créer des ads (IAB 2025).
- **30% des ads vidéo** sont déjà générées ou améliorées par IA ; projection 39% en 2026.
- Le coût de production d'une vidéo AI est passé de ~$50/vidéo (2024) à **$0.05–$5/vidéo** (2026).
- Formats dominants : 9:16 (TikTok/Reels/Shorts), 16:9 (YouTube/CTV), 1:1 (Facebook/Instagram feed).

---

## 2. Approches architecturales

### Tier 1 — Template-based (JSON → Vidéo)
**Principe** : Templates HTML/CSS ou JSON, data injectée via API, rendu headless.

| Outil | Type | Prix | Stack |
|-------|------|------|-------|
| **Remotion** | Open-source | Gratuit (self-host) / $25/mo cloud | React + FFmpeg + Puppeteer |
| **JSON2Video** | SaaS API | $29–$99/mo | REST + JSON templates |
| **Shotstack** | SaaS API | $49–$199/mo | REST + JSON editing API |
| **Creatomate** | SaaS API | $29–$149/mo | REST + templates |

**Best for** : Vidéos branded avec données dynamiques (prix, noms, produits).

### Tier 2 — AI Generation (Prompt → Vidéo)
**Principe** : Modèle génératif text-to-video ou image-to-video.

| Modèle | Prix/sec | Durée max | Qualité | Accès API |
|--------|----------|-----------|---------|-----------|
| **Veo 3.1** (Google) | $0.10–$0.75 | 20s | 4K + audio natif | Vertex AI / fal.ai |
| **Sora 2** (OpenAI) | $0.10–$0.50 | 20s | Cinématique | OpenAI API |
| **Kling 3.0** (Kuaishou) | $0.05–$0.22 | 10–120s | Très bon | fal.ai / direct |
| **Runway Gen-4.5** | $0.50 | 10s | Pro | Runway API |
| **Seedance 2.0** (ByteDance) | $0.15–$0.30 | 10s | Audio+vidéo | ModelArk / fal.ai |
| **Wan 2.6** (Alibaba) | $0.05 | 5s | Bon | fal.ai / open-source |
| **Luma Ray 3** | $0.10 | 5–10s | HDR | Luma API / fal.ai |
| **Pika 2.0** | $0.05–$0.10 | 5–10s | Social-first | Pika API |

**Best for** : Contenu créatif original, variations rapides, tests A/B.

### Tier 3 — Avatar/UGС (Script → Talking Head)
**Principe** : Avatar IA lit un script, fonds dynamiques, voix clonée.

| Outil | Prix | Features |
|-------|------|----------|
| **HeyGen** | $24–$120/mo | Avatars réalistes, 60+ langues |
| **Synthesia** | $22–$67/mo | Avatars pros, lip-sync |
| **Creatify** | $39–$99/mo | URL→vidéo, batch, avatars UGC |
| **Arcads** | $110+/mo | UGC actors IA, e-commerce focus |

**Best for** : Vidéos explicatives, témoignages simulés, démos produits.

---

## 3. Matrice comparative

| Critère | Remotion (self-host) | JSON2Video | Creatify | fal.ai (multi-model) |
|---------|---------------------|------------|----------|----------------------|
| Coût/vidéo | ~$0.01 (compute) | ~$0.50 | ~$2–8 | $0.50–5 |
| Contrôle créatif | Total | Template | Limité | Prompt-only |
| Scale | Illimité | API rate limit | Crédits/mois | Pay-per-use |
| Setup initial | Complexe (DevOps) | Rapide | Immédiat | Rapide |
| Custom branding | ✅ Total | ✅ Partiel | ❌ Limité | ❌ Aucun |
| Voix/audio | ElevenLabs API | ✅ Intégré | ✅ Intégré | ❌ Séparé |
| Self-host possible | ✅ | ❌ | ❌ | ❌ |

---

## 4. Architecture recommandée (pipeline hybride)

```
┌─────────────────────────────────────────────────────────┐
│                    INPUT LAYER                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │ Product   │  │ Brand    │  │ Creative │              │
│  │ Feed/URL  │  │ Assets   │  │ Brief    │              │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘              │
└───────┼──────────────┼──────────────┼────────────────────┘
        │              │              │
        ▼              ▼              ▼
┌─────────────────────────────────────────────────────────┐
│                  ORCHESTRATOR (Go)                       │
│  ┌──────────────────────────────────────────────────┐   │
│  │  • Parse product data (JSON/CSV/URL scrape)       │   │
│  │  • Generate script via LLM (GPT-4o / Claude)     │   │
│  │  • Select template (9:16 / 16:9 / 1:1)           │   │
│  │  • Queue render jobs                              │   │
│  │  • Track status, retry, notify                    │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────────┘
                       │
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
┌────────────┐ ┌──────────┐ ┌──────────────┐
│  RENDER    │ │ AI GEN   │ │   AVATAR     │
│  ENGINE    │ │ (fal.ai) │ │   (HeyGen)   │
│            │ │          │ │              │
│ Remotion + │ │ Veo 3.1  │ │ Script →     │
│ FFmpeg +   │ │ Kling 3  │ │ Talking Head │
│ Headless   │ │ Sora 2   │ │ + Voiceover  │
│ Chrome     │ │          │ │              │
└─────┬──────┘ └────┬─────┘ └──────┬───────┘
      │             │              │
      └─────────────┼──────────────┘
                    ▼
┌─────────────────────────────────────────────────────────┐
│                 POST-PROCESSING                          │
│  • FFmpeg: add watermark, captions, music                │
│  • Generate multi-format (MP4, WebM, GIF)                │
│  • Upload to CDN/S3                                     │
│  • Schedule publish (TikTok, Meta, YouTube API)          │
└─────────────────────────────────────────────────────────┘
```

### Stack technique

| Couche | Technologie | Justification |
|--------|-------------|---------------|
| **Orchestrateur** | Go (stdlib + `net/http`) | Stack existante, performant, binaire unique |
| **Template Engine** | Remotion (Node.js sidecar) | React = réutilisable, animable, testable |
| **AI Vidéo** | fal.ai API (multi-model) | Un seul compte, tous les modèles, pay-per-use |
| **Voix/Audio** | ElevenLabs API | Meilleure qualité TTS multilingue |
| **Scripts** | OpenAI/Claude API | Génération de scripts publicitaires |
| **Post-process** | FFmpeg (system call) | Watermark, captions, format conversion |
| **Stock footage** | Pexels/Pixabay API | B-roll gratuit |
| **Stock music** | Pixabay Music / Uppbeat API | Musique libre de droits |
| **Storage/CDN** | S3-compatible (Wasabi $7/TB) | Moins cher qu'AWS S3 |
| **Queue** | SQLite (existing) ou Redis | Job queue simple |
| **Scheduler** | Cron / systemd timer | Déclenchement périodique |

---

## 5. Modèle économique pour le client

### Option A — SaaS self-serve
- **$49/mo** : 10 vidéos/mois
- **$99/mo** : 30 vidéos/mois  
- **$249/mo** : 100 vidéos/mois + API access

### Option B — Full service (agence)
- **$500–2000/mo** : Pipeline géré + stratégie créative + rapports
- Marge brute : ~70% (coûts API ~$0.50–3/vidéo)

### Coûts mensuels estimés (1000 vidéos)
| Poste | Coût |
|-------|------|
| fal.ai (Veo 3.1 Fast, 500 vidéos × $0.50) | $250 |
| ElevenLabs (500k chars/mo) | $22 |
| OpenAI (scripts, 1M tokens) | $10 |
| Wasabi (storage + egress) | $15 |
| Serveur VPS (8 vCPU, render) | $30–60 |
| **Total** | **~$350/mo** |

---

## 6. Roadmap d'implémentation

### Phase 1 — MVP (2 semaines)
- [ ] Go orchestrator : file watcher + job queue SQLite
- [ ] Remotion sidecar : 3 templates (ad 9:16, ad 16:9, product showcase)
- [ ] Script generation via OpenAI
- [ ] Render pipeline : Remotion → FFmpeg → S3
- [ ] Simple web UI : upload CSV → voir résultats

### Phase 2 — AI Generation (1 semaine)
- [ ] Intégration fal.ai (Veo 3.1, Kling 3.0)
- [ ] Pipeline hybride : template Remotion + clips AI en overlay
- [ ] ElevenLabs TTS intégré
- [ ] Auto-captions via Whisper

### Phase 3 — Distribution & Scale (1 semaine)
- [ ] Multi-plateforme export (TikTok, Meta, YouTube formats)
- [ ] Direct publishing API (Meta Graph, TikTok Content Posting)
- [ ] A/B testing : générer 5 variations, tracker CTR
- [ ] Dashboard analytics

---

## 7. Fichiers clés du projet

```
video-pipeline/
├── cmd/
│   ├── orchestrator/    # Go main — job queue, API server
│   └── worker/          # Go worker — render jobs
├── internal/
│   ├── templates/       # Remotion React templates
│   ├── generator/       # Script generation (OpenAI)
│   ├── renderer/        # Remotion + FFmpeg wrapper
│   ├── publisher/       # Social media APIs
│   └── storage/         # S3 upload/download
├── remotion/            # Node.js Remotion project
│   ├── src/
│   │   ├── templates/   # AdSpot, ProductShowcase, UGC
│   │   └── components/  # Reusable: LowerThirds, CTAs, etc.
│   └── render.ts        # CLI render entrypoint
├── web/                 # Simple admin UI (Go + Unpoly)
└── data/                # SQLite DB
```

---

## 8. Risques et mitigations

| Risque | Probabilité | Mitigation |
|--------|-------------|------------|
| Coûts API qui explosent | Moyenne | Rate limiting, cache des renders, fallback template-only |
| Qualité AI inconsistante | Haute | Human-in-the-loop approval, A/B testing auto |
| Remotion render lent | Haute | Lambda/GPU spot instances, pre-render cache |
| Droits musique/images | Moyenne | APIs stock certifiées, vérification automatique |
| Changement prix API | Haute | Architecture multi-provider, fallback open-source (Wan 2.6) |

---

## 9. Sources

- IAB 2025 Video Ad Spend & Strategy Report
- Artificial Analysis Video Arena (classement modèles)
- Creatify, HeyGen, Synthesia pricing pages (2026)
- fal.ai API documentation
- Remotion.dev documentation
- JSON2Video, Shotstack API documentation
- n8n community workflows
- buildmvpfast.com — AI video API pricing comparison

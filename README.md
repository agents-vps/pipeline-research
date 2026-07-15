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

---

## 10. Performance benchmarks (real data)

### AI vs human creative
| Métrique | AI vs Humain | Contexte |
|----------|-------------|----------|
| CTR (Meta) | **+12%** | Dataset de 50 000+ variations d'ads |
| Conversion ($100+ AOV) | **−8%** | Produits chers = confiance humaine nécessaire |
| Conversion ($500+ AOV) | **−14%** | Écart qui se creuse avec le prix |
| ROAS parity | **AOV < $100** | Pour produits abordables, l'IA égale l'humain |
| CPA (Meta Advantage+) | **−32%** | Campagnes optimisées par IA |
| ROAS (YouTube AI) | **+17%** | Nielsen 2025 |

**Conclusion** : Pour e-commerce < $100 AOV, l'IA est déjà supérieure. Pour luxe/considération, mixer IA (volume/testing) + humain (finishing).

### Pourquoi l'IA performe
- Volume de testing : 10-50x plus de variations qu'une équipe humaine
- Itération rapide : un test qui prenait 2 semaines → 2 heures
- Algorithmes de plateforme (Meta, TikTok) qui récompensent la diversité créative

---

## 11. Voix & Audio : Qwen3-TTS (game changer)

**Jusqu'ici** : ElevenLabs était le standard (qualité > prix).  
**Maintenant** : Qwen3-TTS change la donne.

| Caractéristique | ElevenLabs | Qwen3-TTS |
|-----------------|------------|-----------|
| **Licence** | Propriétaire | Apache 2.0 (open-source) |
| **Coût** | $0.30/1k chars | $0 (self-host, coût GPU uniquement) |
| **Voice cloning** | ✅ Premium | ✅ 3 secondes d'audio |
| **Voice design** | ❌ | ✅ (prompt texte → voix) |
| **Langues** | 29 | 10 (EN, FR, ES, DE, JP, KR, ZH, RU, PT, IT) |
| **Modèle** | 1.7B params | 1.7B params (4.54 GB) |
| **VRAM min** | N/A | RTX 3090 (24 GB) |
| **Streaming** | ✅ | ✅ ultra-low latency |
| **Self-host** | ❌ | ✅ (Docker, Ubuntu) |
| **Déploiement** | API cloud | GPU local ou VPS GPU (~$0.50/h) |

**Impact coût pipeline** : ElevenLabs $22/mo → Qwen3-TTS $0 (ou ~$15/mo GPU partagé). Économie de ~$200-500/an.

---

## 12. Remotion SSR : options de déploiement server-side

| Option | Vitesse | Coût | Scale | Setup |
|--------|---------|------|-------|-------|
| **Remotion Lambda** | Rapide (distribué) | Pay-per-render | Auto-scale | 1 commande |
| **Vercel Sandbox** | Rapide | Inclus Vercel | Limité | Le + simple |
| **Cloud Run** | Bon | Pay-per-second | Auto-scale | Dockerfile |
| **Serveur custom (VPS)** | Lent (1 machine) | Fixe ~$30-60/mo | Manuel | Complexe |
| **GitHub Actions** | Très lent | Gratuit (limité) | 6h/mois gratuit | Simple |

**Recommandé** : Remotion Lambda pour la prod, VPS pour le dev/test.

### Nouvelle alternative : Revideo
- Fork open-source de Motion Canvas
- Ajoute rendering API + templates + server-side
- Package `@revideo/core` (~3K npm downloads/semaine)
- Plus léger que Remotion si on n'a pas besoin de React

---

## 13. Paysage concurrentiel (video ad SaaS)

| Outil | Positionnement | Prix | Cible |
|-------|---------------|------|-------|
| **Creatify** | URL→vidéo, avatars UGC | $39-99/mo | E-commerce DTC |
| **HeyGen** | Avatar UGC pro | $24-120/mo | Marketing B2B/B2C |
| **Synthesia** | Talking head corporate | $22-67/mo | Formation, corpo |
| **Arcads** | UGC actors IA | $110+/mo | E-commerce premium |
| **Rocketium** | Creative automation retail | Sur devis | Grande distribution |
| **Hunch** | DPA/catalog ads Meta | Sur devis | Mid-market e-commerce |
| **Marpipe** | Feed-to-creative testing | Sur devis | E-commerce performance |
| **JSON2Video** | API JSON→vidéo | $29-99/mo | Développeurs |
| **Shotstack** | API video editing | $49-199/mo | Développeurs |

**Notre angle différenciateur** : Pipeline hybride (template + AI gen + avatar) avec self-host possible (Qwen3-TTS + Remotion open-source) = coût marginal quasi nul.

---

## 14. Recommandation mise à jour

### Stack finale recommandée

| Couche | Choix | Raison |
|--------|-------|--------|
| **Template rendering** | Remotion (open-source) | Gratuit, React, mature |
| **AI vidéo** | fal.ai (multi-model) | Pay-per-use, tous les modèles |
| **Voix/Audio** | **Qwen3-TTS** (self-host) | Gratuit Apache 2.0, qualité pro |
| **Script ads** | Claude/GPT-4o API | Création de scripts marketing |
| **Post-process** | FFmpeg (system call) | Gratuit, tout faire |
| **Stock media** | Pexels/Pixabay (gratuit) | B-roll + musique |
| **Renderer cloud** | Remotion Lambda | Scale auto, pay-per-render |
| **Orchestrateur** | Go (binaire unique) | Stack maîtrisée |

### Coût marginal par vidéo (estimé)
- fal.ai (5s Kling 3.0) : $0.10
- Qwen3-TTS : $0 (self-host)
- Script LLM : $0.002
- Remotion Lambda render : ~$0.02
- **Total : ~$0.12/vidéo** 🎯

À ce prix, on peut générer **1000 vidéos pour ~$120** et les revendre $5-50/vidéo.

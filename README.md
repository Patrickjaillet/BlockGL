# BlockGL

> Éditeur visuel de shaders GLSL · Windows 10/11 · Offline · Zéro IA

BlockGL permet de composer des fragment shaders compatibles **Shadertoy (onglet Image)** par glisser-déposer de nœuds, avec prévisualisation WebGL2 temps réel.

---

## Prérequis

| Outil | Version minimale | Lien |
|-------|-----------------|------|
| Node.js | 20 LTS | https://nodejs.org |
| pnpm | 9+ (optionnel) | `npm i -g pnpm` |
| Rust | stable (1.77+) | https://rustup.rs |
| WebView2 | Runtime installé | Pré-installé sur Win10 21H2+ et Win11 |

> **Windows uniquement.** Le projet ne cible pas macOS/Linux.

---

## Installation

### Option A — pnpm (recommandé)

```bash
# Installer pnpm si pas encore fait
npm i -g pnpm

# Depuis le dossier BlockGL
pnpm install
pnpm tauri dev
```

### Option B — npm

Le fichier `.npmrc` inclus configure `legacy-peer-deps=true` automatiquement.

```bash
npm install
npm run tauri dev
```

> Si npm affiche malgré tout une erreur ERESOLVE :
> ```bash
> npm install --legacy-peer-deps
> ```

### Vérifier Rust avant de commencer

```bash
rustup update stable
rustc --version   # 1.77+ requis
```

La première compilation Rust peut prendre 2-3 minutes. Les suivantes sont incrémentales.

---

## Scripts disponibles

| Commande (pnpm) | Commande (npm) | Description |
|-----------------|----------------|-------------|
| `pnpm tauri dev` | `npm run tauri dev` | Démarre l'app Tauri en mode hot-reload |
| `pnpm dev` | `npm run dev` | Lance uniquement Vite (preview navigateur, sans Tauri) |
| `pnpm build` | `npm run build` | Build de production du front |
| `pnpm tauri build` | `npm run tauri build` | Build complet → MSI dans `src-tauri/target/release/bundle/msi/` |
| `pnpm test` | `npm test` | Tests unitaires Vitest |
| `pnpm lint` | `npm run lint` | ESLint sur tout `src/` |
| `pnpm format` | `npm run format` | Prettier sur tout `src/` |

---

## Structure du projet

```
blockgl/
├── src/                        # Code TypeScript / React
│   ├── core/                   # Moteur de graphe pur (Node, Edge, GraphEngine, nodeDefs)
│   │   ├── compiler.worker.ts  # Compilation GLSL hors thread UI (Web Worker)
│   │   └── __tests__/          # Tests Vitest
│   ├── renderer/
│   │   ├── canvas/             # Éditeur de graphe Canvas 2D (Viewport, Renderer, Controller)
│   │   ├── WebGLRenderer.ts    # Rendu temps réel du fragment shader (preview)
│   │   └── testShader.ts       # Shader de test Phase 0
│   ├── ui/                     # Composants React (GraphEditor, NodePalette, Inspector, MenuBar, SplashScreen…)
│   ├── store/                  # État global Zustand (graphStore, historySlice)
│   ├── io/
│   │   ├── projectIO.ts        # Ouvrir/sauvegarder/exporter — Tauri natif ou repli navigateur
│   │   └── autosave.ts         # Autosave 60s (localStorage + sidecar .autosave sous Tauri)
│   ├── tauri/
│   │   └── bridge.ts           # Wrappers typés sur les commandes Tauri
│   ├── styles/
│   │   └── globals.css         # Design tokens + reset global
│   ├── App.tsx                 # Shell applicatif (palette + graphe + preview)
│   └── main.tsx                # Point d'entrée React
│
├── src-tauri/                  # Code Rust / Tauri
│   ├── src/
│   │   ├── main.rs             # Point d'entrée Tauri
│   │   └── commands.rs         # Commandes : save, load, export, screenshot, clipboard
│   ├── Cargo.toml
│   ├── build.rs
│   └── tauri.conf.json         # Config Tauri (Windows MSI, associations .blockgl)
│
├── .npmrc                      # legacy-peer-deps=true (compatibilité npm)
├── index.html
├── vite.config.ts
├── tsconfig.json
├── vitest.config.ts
└── ROADMAP.md
```

---

## Uniforms Shadertoy supportés

BlockGL injecte automatiquement ces uniforms dans chaque shader. Ils sont disponibles dans `mainImage()` sans aucune déclaration manuelle.

| Uniform | Type GLSL | Description |
|---------|-----------|-------------|
| `iTime` | `float` | Secondes depuis le démarrage |
| `iTimeDelta` | `float` | Durée de la frame précédente |
| `iFrame` | `int` | Compteur de frames |
| `iResolution` | `vec3` | `vec3(width, height, 1.0)` en pixels |
| `iMouse` | `vec4` | `xy` = position souris, `zw` = position au dernier clic |
| `iDate` | `vec4` | `x`=année, `y`=mois (0–11), `z`=jour, `w`=secondes dans la journée |

> **Hors scope v1.0 :** `iChannel0..3`, buffers A/B/C/D.

---

## Format de fichier `.blockgl`

```json
{
  "version": "1.0",
  "meta": { "name": "My Shader", "created": "2026-06-20", "modified": "2026-06-20", "tags": [] },
  "graph": {
    "nodes": [
      { "id": "n1", "type": "UVNode", "position": [100, 200], "params": {} },
      { "id": "n_out", "type": "OutputNode", "position": [600, 260], "params": {} }
    ],
    "edges": [{ "from": "n1:out", "to": "n_out:fragColor" }]
  },
  "preview": { "speed": 1.0, "resolution": "720p", "paused": false }
}
```

---

## Phases de développement

| Phase | Semaines | Statut |
|-------|----------|--------|
| **0** — Fondations (ce repo) | S1-S2 | ✅ |
| **1** — Moteur de graphe | S3-S6 | ✅ |
| **2** — Bibliothèque de nœuds | S7-S9 | ✅ 139 nœuds |
| **3** — Preview temps réel | S10-S11 | ✅ |
| **4** — UI/UX finale | S12-S14 | ✅ |
| **5** — Export & I/O | S15-S16 | ✅ |
| **6** — Polish & packaging | S17-S19 | ✅ |

Voir [`ROADMAP.md`](./ROADMAP.md) pour le détail complet.

La v1.0 est un outil pour utilisateurs à l'aise avec les graphes de nœuds et le GLSL. [`ROADMAP2.md`](./ROADMAP2.md) planifie un **Mode Simple** (cartes illustrées, curseurs visuels, zéro jargon) pour rendre la création de shaders accessible à n'importe qui — y compris un enfant — sans toucher au Mode Pro existant.

---

*BlockGL v1.0 — Visual GLSL Shader Editor · Juin 2026*

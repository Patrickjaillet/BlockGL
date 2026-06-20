# Changelog

## 1.0.2

- Correction d'ESLint : il manquait `eslint.config.js` (format flat config requis par ESLint v9) et `eslint-plugin-react-refresh` référencé dans l'ancien `.eslintrc.cjs` n'était jamais installé — `pnpm lint` échouait systématiquement depuis le début du projet. Nouvelle config flat minimale (`@eslint/js`, `@typescript-eslint`, `react-hooks`) ; lint maintenant vert sur tout `src/`.
- Correction de `boundingBox()` dans `graphStore.ts` (utilisée par le groupement et le collage) : elle utilisait une taille de nœud fixe (180×80) au lieu des helpers réels `nodeWidth`/`nodeHeight`, ce qui pouvait tronquer le cadre d'un `Group / Frame` créé autour de nœuds à plusieurs ports (ex. Custom GLSL, 4 entrées).
- `groupSelected()` sélectionne désormais le nœud `Group / Frame` nouvellement créé, comme le font déjà l'ajout, la duplication et le collage de nœuds.
- Correction de `GraphEngine.validate()` : l'avertissement "nœud non connecté" se déclenchait systématiquement sur les nœuds purement organisationnels sans aucun port (Comment, Group/Frame), qui ne peuvent par construction jamais être connectés.
- Audit complet : `tsc`, `cargo check`/`cargo clippy`, 381 tests Vitest + 7 tests Rust, build de production, et les 139 types de nœuds compilés ensemble sur WebGL2 réel (et non simulé) — tous verts, aucune autre anomalie détectée.

## 1.0.1

- Correction d'un défaut visuel au lancement : le graphe de départ est vide tant qu'aucun template n'est choisi sur l'écran d'accueil, ce qui déclenchait l'erreur "nœud Output manquant" — visible par transparence à travers l'overlay du splash screen (badge dans la barre de statut et message d'erreur en surimpression de la preview). L'affichage de cette erreur est maintenant suspendu tant que le splash screen est ouvert.

## 1.0.0

- Compilation du graphe vers GLSL déplacée dans un Web Worker dédié (`compiler.worker.ts`) : le thread UI ne bloque plus jamais, même sur un graphe volumineux ; requêtes debouncées 200ms avec id de requête pour ignorer les réponses obsolètes.
- Virtualisation du rendu Canvas 2D : les nœuds et arêtes hors du rectangle visible (+ marge) ne sont plus dessinés. Mesuré sur un graphe de 321 nœuds / 319 connexions : ~59 fps en idle, ~48 fps en pan actif.
- Autosave toutes les 60 secondes (`localStorage`, plus un fichier sidecar `<chemin>.autosave` sous Tauri quand un chemin de projet existe) avec bannière de récupération au démarrage si une sauvegarde non enregistrée est détectée.
- Message d'erreur enrichi quand WebGL2 est indisponible (pilote obsolète, accélération matérielle désactivée, VM/RDP sans GPU, lien vers `edge://gpu`).
- `parseProject` rejette désormais un fichier `.blockgl` dont la version majeure est supérieure à la version supportée, tout en restant tolérant aux versions mineures ou antérieures.
- 9 nouveaux tests Vitest (validation de version) et 7 nouveaux tests `cargo test` (round-trip save/load, export GLSL/HTML, flip vertical PNG, clipboard) — 380 tests Vitest et 7 tests Rust au total, tous verts.
- Les 8 templates de démarrage vérifiés via WebGL2 réel (navigateur Chromium, pas de mock) : compilation sans erreur driver pour chacun.
- Build de production vérifié (`pnpm build`, `pnpm tauri build`) : installeur MSI de 2,37 Mo (cible < 20 Mo), association de fichier `.blockgl` et icône multi-résolutions confirmées dans le bundle WiX généré.
- Correction de l'identifiant de bundle (`com.blockgl.app` → `com.blockgl.editor`, évite le conflit d'extension `.app` côté macOS signalé par le bundler).
- Versions Cargo.toml et package.json synchronisées à `1.0.0`.

## 0.12.0

- Format de fichier `.blockgl` (`src/core/project.ts`) : version, métadonnées (nom, dates de création/modification, tags), graphe, réglages de preview — sérialisation/parsing avec validation structurelle et messages d'erreur clairs (`ProjectParseError`).
- Nouvelles commandes Tauri `open_file_dialog`/`save_file_dialog` (crate `tauri-plugin-dialog`) ; les commandes `save_project`/`load_project`/`export_glsl`/`export_html`/`save_screenshot`/`copy_to_clipboard` existaient déjà depuis la Phase 0 mais n'étaient câblées nulle part — c'est fait.
- Couche `src/io/projectIO.ts` : chaque opération (ouvrir/sauvegarder/exporter GLSL/exporter HTML/copier) détecte l'environnement Tauri et bascule sur un repli navigateur (téléchargement, sélecteur de fichier) — testable et fonctionnel en `pnpm dev` comme en build packagé.
- 8 templates de démarrage (Blank, UV Debug, Plasma Wave, Perlin Landscape, SDF Shapes, Voronoi Cells, Glitch TV, Kaleidoscope), construits par code à partir des nœuds existants, testés automatiquement (chacun doit compiler en GLSL valide).
- Splash screen au lancement (et via `Ctrl+N`) présentant les 8 templates en grille, plus "Ouvrir un projet existant…".
- Menu File/Export entièrement câblé : Nouveau (`Ctrl+N`), Ouvrir (`Ctrl+O`), Sauvegarder (`Ctrl+S`), Sauvegarder sous (`Ctrl+Shift+S`), Exporter GLSL (`Ctrl+E`), Exporter HTML, Copier le GLSL.
- 27 nouveaux tests (format de projet, actions `loadGraph`/`newProject`, compilation de chaque template). Vérifié de bout en bout en navigateur (cycle complet créer → sauvegarder → recharger → exporter, sans erreur console) et compilation Rust réelle validée (`cargo check` + `pnpm tauri dev`).

## 0.11.0

- Barre de menu File/Edit/View/Export/Help (Edit/View/Help pleinement fonctionnels ; File/Export listent leurs actions en attente de la Phase 5).
- Recherche rapide de nœud façon Blender (`Tab`) : palette flottante, navigation clavier, insertion au centre du viewport.
- Inspecteur enrichi : sliders scrubbables (`DragNumberInput`, clic-glisser horizontal), color picker natif pour les nœuds Couleur, dropdowns pour les paramètres à choix discrets (nouveau mécanisme `paramOptions`), éditeur `Custom GLSL` avec coloration syntaxique (mots-clés/types/nombres/commentaires, sans dépendance externe).
- Rendu visuel du nœud `Group / Frame` : cadre englobant dessiné derrière les nœuds qu'il contient, clic transparent sur son corps (seul le bandeau de titre est cliquable), redimensionnable via les params `width`/`height`.
- Palette : icônes par catégorie (`lucide-react`), surlignage du terme recherché, double-clic qui centre désormais sur le viewport réel (corrige un bug : posait toujours le nœud à l'origine 0,0).
- Raccourcis ajoutés : `G` (grouper la sélection), `Ctrl+C`/`Ctrl+V` (copier/coller).
- Panneau "Aide rapide" (raccourcis + concepts) et badge "Nœud terminal manquant" dans la barre de statut.
- 6 nouveaux tests (`nodeWidth`/`nodeHeight`/`hitTestNode` pour Group/Frame). Vérifié visuellement de bout en bout : groupement, copier-coller, recherche rapide, dropdowns, color picker, menu Edit, surlignage de recherche.

## 0.10.0

- Recompilation automatique du graphe vers la preview WebGL (debounce 200ms), remplaçant le chargement manuel.
- Surlignage en rouge du nœud responsable d'une erreur de compilation GLSL, avec le message précis du driver (numéro de ligne) affiché dans le panneau et la barre de statut — fonctionne aussi bien pour les erreurs de validation structurelle (cycle, type, Output manquant) que pour les erreurs de compilation driver réelles.
- Le compilateur (`compileGraph`) sépare désormais `mainImageGLSL` (pour le renderer, sans uniforms/`#version`) du `glsl` autonome (affichage/export), avec une source map ligne → nœud (`nodeForLine`). **Corrige un bug latent** : l'ancien flux "Charger dans la preview" envoyait un GLSL avec ses propres déclarations `uniform` dans un renderer qui les préfixait déjà, provoquant une redéfinition jamais testée bout-en-bout.
- Contrôles de preview étendus : vitesse 0.1×-10× (était 0.1×-4×), sélecteur de résolution interne (360p/540p/720p/1080p/native, découplée de la taille CSS), snapshot PNG jusqu'à 4K (rendu offscreen, encodage via canvas 2D, téléchargement), mode plein écran in-app (CSS, barre de contrôle flottante, `F11`/`Échap`).
- Correction du scaling souris → résolution interne du canvas (nécessaire dès que la résolution diffère de la taille affichée).
- 8 nouveaux tests (source map ligne→nœud, offset de ligne du renderer).

## 0.9.0

- 9 nœuds Logique & Utilitaires (§2.8) : If/Select, Compare, And, Or, Not, Custom GLSL, Comment, Group/Frame, Preview Pin. **Clôture la bibliothèque de nœuds de la Phase 2 (139 nœuds, §2.1-2.8).**
- Nouveau mécanisme de compilation `emitBlock` : permet à un nœud d'injecter un bloc GLSL multi-instructions scopé (`{ }`) plutôt qu'une simple expression — utilisé par `Custom GLSL` (4 entrées `vec3` a/b/c/d, variable `result`), réutilisable pour de futurs nœuds complexes.
- `Comment`, `Group / Frame` et `Preview Pin` sont des nœuds purement organisationnels qui se posent et se sérialisent sans affecter le GLSL généré ; leur rendu visuel avancé (cadre redimensionnable, lecture live GPU) est différé aux Phases 4 et 3 respectivement.
- 8 nouveaux tests ciblés (opérateurs Compare, scoping Custom GLSL multi-instances, nœuds non compilés). Vérifié visuellement : bloc Custom GLSL personnalisé combiné à Compare + If/Select.

## 0.8.0

- 13 nœuds Effets & Post-process (§2.7) : Vignette, Chromatic Aberration, Film Grain, Glitch, Scanlines, Kaleidoscope, Polar/Cartesian Coords, Pixelate, Barrel Distortion, Dithering, Edge Detect, Sharpen (130 nœuds au total).
- Faute de buffers/`iChannel`, les effets image-space classiques sont réinterprétés en transforms UV procéduraux ou en nœuds à échantillons multiples explicites (voir note ROADMAP §2.7) plutôt que d'échantillonner une image inexistante.
- **Correctif compilateur** : les sorties multi-valeurs composées (ex. `uv + vec2(...)` pour Chromatic Aberration) n'étaient pas parenthésées avant réinjection dans une expression consommatrice, causant un bug de précédence d'opérateurs (`a + b / c` au lieu de `(a + b) / c`). Trouvé par un test ciblé, corrigé dans `compileGraph`.
- 5 nouveaux tests ciblés. Vérifié visuellement : vignette, scanlines et dithering Bayer 4×4 rendent correctement.

## 0.7.0

- 15 nœuds Noise & Patterns (§2.6) : Hash 1D/2D/2D→2D, Value/Gradient/Simplex Noise 2D, FBM, Domain Warp, Voronoi (+Smooth), Checker, Stripes, Hexagonal Grid, Truchet Tiles, Dots Grid. Franchit le seuil "60+ nœuds" du livrable Phase 2 (117 nœuds au total).
- 13 nouvelles fonctions GLSL helper, dont une chaîne de dépendances à 2 niveaux (`hash21` → `valueNoise2D` → `fbm2D`) et des surcharges de fonction (`permute289(float)` / `permute289(vec3)`) pour le bruit simplex.
- 8 nouveaux tests ciblés (dédup multi-niveaux, ordre des surcharges, sorties packées vec2 pour Voronoi/Hexagonal Grid). Vérifié visuellement : FBM nuageux et cellules de Voronoï correctement formées en rendu réel.

## 0.6.0

- 18 nœuds SDF (§2.5) : Circle, Box 2D, Rounded Box, Line Segment, Ring, Triangle, Polygon, Union/Subtract/Intersect (durs et lissés), Repeat, Repeat Limited, Elongate, Onion, To Color.
- 8 nouvelles fonctions GLSL helper pour les SDFs (`sdBox2D`, `sdRoundBox2D`, `sdSegment`, `sdEquilateralTriangle`, `sdRegularPolygon`, `sminSDF`, `ssubSDF`, `sintSDF`).
- 12 nouveaux tests ciblés (formules, dédup des opérateurs lissés, transform de position pour Repeat) en plus du test de compilation systématique par nœud. Vérifié visuellement : union lissée cercle + rectangle.

## 0.5.0

- 15 nœuds Couleurs (§2.4) : Color Picker (+Alpha), HSV↔RGB, Hue Shift, Gamma Correct, Linear to sRGB, Tone Map (ACES/Reinhard), Invert Color, Blend (Mix/Add/Multiply/Screen), Contrast/Brightness.
- Le compilateur supporte l'injection de fonctions GLSL helper réutilisables (`src/core/glslHelpers.ts`) : déduplication par graphe, ordre topologique garanti, utilisé par `RGB to HSV` et `Hue Shift`.
- 15 nouveaux tests ciblés sur l'injection de helpers (dédup, ordre, position dans le fichier généré) en plus du test de compilation systématique par nœud.

## 0.4.0

- Bibliothèque de nœuds étendue à 69 types : Inputs (15, dont UV Centered, TimeDelta, Mouse Position, Frame, Date, constantes Vec2/Vec4/Int/Bool), Maths (40, tous les built-ins GLSL standards : trigonométrie, puissances, interpolation, clamp/min/max...), Vecteurs (13 : Compose/Swizzle/Split Vec2-4, Remap, Rotate 2D, Reflect, Refract, Face Forward).
- Le compilateur (`compileGraph`) supporte désormais les nœuds multi-sorties (`emit()` peut retourner une table `{ portId: expression }`), utilisé par `Split Vec2/3/4`.
- 74 nouveaux tests : un test de compilation par définition de nœud, plus des cas ciblés (multi-sorties, swizzle paramétré, formules Remap/Negate/Reciprocal, littéraux Int/Bool).

## 0.3.0

- Compilateur Graphe → GLSL (`src/core/compiler.ts`) : tri topologique, génération de variables temporaires typées, injection automatique des uniforms Shadertoy, sortie `mainImage()`.
- Panneau "Graphe → GLSL" dans l'UI : affichage en direct du GLSL compilé (ou des erreurs de validation), bouton pour charger le résultat dans la preview WebGL.

## 0.2.0

- Éditeur de graphe interactif : glisser-déposer de nœuds depuis la palette, tirage de fils entre ports avec validation de type en direct, sélection (clic, lasso, multi-sélection additive), déplacement de nœuds avec snap-to-grid, double-clic pour ouvrir l'inspecteur, clic droit pour dupliquer/supprimer.
- Registre minimal de 12 types de nœuds (UV, Time, Resolution, Mouse, constantes, Add, Multiply, Mix, Sin, Compose Vec3, Output).
- Store Zustand du graphe avec historique undo/redo et sélection.
- Raccourcis clavier : `Ctrl+Z` / `Ctrl+Y`, `Ctrl+A`, `Delete`, `D` (dupliquer), `Escape`, `F` (cadrer la sélection), `H` (réinitialiser la vue).
- Correction du fichier `tauri.conf.json` (schéma Tauri v2 : `identifier` racine, `fileAssociations`).
- Génération des icônes de l'application (`pnpm tauri icon`).
- Correction d'un import cassé dans `phase0.test.ts`.

## 0.1.0

- Initialisation du projet (Tauri 2 + Vite + React + TypeScript).
- Proof of concept WebGL2 : rendu d'un shader de test avec uniforms Shadertoy (`iTime`, `iResolution`, `iMouse`, etc.).
- Moteur de graphe TypeScript pur (`GraphEngine`) : mutations, validation, tri topologique.
- Canvas d'édition 2D : viewport (pan/zoom), rendu des nœuds et arêtes, minimap, sélection rectangulaire.
- Pile undo/redo immuable (`HistoryStack`).

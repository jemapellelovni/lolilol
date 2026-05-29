# 08 — Contexte divers (MaSurvie V2)

Repères transverses utiles à une IA qui crée du contenu : où vivent les fichiers, conventions du projet, cheat-sheet des commandes, et pièges connus.

> **Source du code** : `D:\MaSurvie V2\plugins custom\`. **Repo public** : aucun secret (tokens, webhooks, identifiants MySQL, clés API) ne doit jamais figurer dans le contenu produit.

---

## 1. Emplacements clés des fichiers

### Côté plugin (Minecraft, Paper/Purpur 1.21, Java 21)

La plupart des plugins ont été **absorbés en modules de `SurvivorCore`** (DiscordLink, ReactionTracker, Horde, Radiation, StreamCode, LootRegion, RegionStats, Territory, Convoy, ZoneSpawn, ItemModule, AxTrade, Tickets…).

| Fichier / dossier | Rôle |
| --- | --- |
| `plugins/SurvivorCore/config.yml` | Config principale + activation des modules (`modules.<nom>: true`). |
| `plugins/SurvivorCore/bdd.yml` | **Config base de données — SECRET, non divulgué.** Volontairement exclu de l'édition par le panel. Migration auto depuis `config.yml`. Ne jamais y mettre de `storage:` dans `config.yml`. |
| `plugins/SurvivorCore/lang_fr.yml` | **Tous les messages** des modules, sections namespacées (horde/radiation/streamcode/lootregion/regionstats/territory/convoy…). ~70 KB. Point d'accès code : `LangManager.getModuleString(mod, key)`. Plus de `messages.yml` par module. |
| `plugins/RadiationCore/zones.yml` | Niveaux de radiation par région WG (`radiation_level`, `visual_intensity`). Auto-généré, jamais écrasé. |
| `plugins/SurvivorCore/timeslots.yml` | Créneaux dynamiques (Nuit Creuse, Rush Weekend). |
| `plugins/SurvivorCore/schedules.yml` | Tâches planifiées (ex. bilan narrateur à 22h). |
| `plugins/WebhookReport/webhooks.yml` | URLs de webhooks Discord (**SECRET**) + couleurs/titres/fields des embeds de report. |
| `deploy/` | **Jars de prod** — fait foi. Après chaque build, lancer `update-deploy.ps1`. |

### Côté bot Discord (`bot/`, Node.js + discord.js)

| Chemin | Rôle |
| --- | --- |
| `bot/.env` | **Tous les secrets bot** (token, clés API Anthropic, tokens partagés, URLs webhooks). Non divulgué. Pas de commentaire inline (mettre `# …` sur la ligne AVANT la variable). |
| `bot/src/modules/embeds/` | Builder d'embeds + composants (boutons, menus). |
| `bot/src/modules/mcAnnounce/` | Annonces déclenchées depuis le jeu (`mc.announce`). |
| `bot/src/modules/welcome/` | Message de bienvenue (public + DM). |
| `bot/src/modules/autoreact/` `autorole/` | Réactions auto + auto-rôles. |
| `bot/src/modules/narrator/` | Narrateur IA (personas, mémoire 24h, embeds, coût). |
| `bot/src/modules/webhookbridge/` | Pont WebhookReport ↔ Discord (réactions, fils, notifs). |
| `bot/src/modules/discordlink/` `reactiontracker/` `tickets/` | Liaison comptes, suivi réactions, tickets. |

### Côté panel web (`survivor-panel/`, Node 20+, Fastify)

| Chemin | Rôle |
| --- | --- |
| `survivor-panel/.env` | **Secrets panel** (OAuth Discord, clés de session/chiffrement, MySQL, Pterodactyl). Non divulgué. `ENCRYPTION_KEY` ne change JAMAIS après activation 2FA. |
| `survivor-panel/SETUP.md` | Procédure d'installation détaillée. |
| `survivor-panel/uploads/embeds/` | Images uploadées pour les embeds. |

### Wiki (source de vérité documentaire)

`wiki/` (Markdown) + `wiki/<plugin>/README.html`|`API.html` (versions HTML à tenir à jour). `wiki/concept_masurvie.md` = page concept à lire en premier. `wiki/index.html` = index avec dates de modif.

---

## 2. Conventions transverses

- **Langue : français partout** (messages joueurs, annonces, lore).
- **MiniMessage** pour les couleurs/formats en jeu (`<#FACC15>`, `<light_purple>`, etc.) dans les YAML de lang/loot.
- **Base de données partagée** : une seule instance MySQL/MariaDB (Minestrator) pour le plugin, le bot et le panel. Tables préfixées : `mb_*` (bot : embeds, autoreact, autorole, mc_announce, welcome, link), `panel_*` (panel), `reaction_*`, `region_*`, `account_links`, `pending_links`, `stream_*`, `zonespawn_stats`, `pocket_contents`, etc. Convention : **temps = BIGINT epoch-ms** (jamais `TIMESTAMP`/`NOW()` côté DiscordLink).
- **API HTTP/WS mutualisée** : un seul `CoreHttpService` (port `25580`), auth **Bearer** partagée (pas de `X-API-Key`, pas de port par module). Le bot ↔ plugin communiquent via REST + WebSocket sur ce service.
- **Plugins découplés** : tout passe par **MaSurvieEventHub** (un listener par source → N sinks, normalisé en `HubEvent`). Personne ne dépend de personne en dur.
- **Ordre de chargement** : configs jamais écrasées par Bukkit/Bukkit ne remplace pas un fichier existant → après refacto lang/config, **diff + copie manuelle** côté serveur (drift entre `deploy/` et le serveur réel).
- **Prod = serveur DISTANT** (upload FileZilla/FTP) ; `D:\server\` local n'est PAS la prod. Penser au cache `plugins/.paper-remapped/<jar>`.
- **Git absent** sur la machine : rollback via snapshots robocopy.

---

## 3. Cheat-sheet des commandes

### Joueur — survie / radiation

| Commande | Effet |
| --- | --- |
| `/link` / `/unlink` | Lier / délier son compte Discord (code 6 chiffres). |
| `/radiation` | État radiation (dose, protection, signature). |
| `/geiger` | Compteur Geiger (niveau de zone, hotspots). |
| `/belt` | Ceinture d'équipement radiation (filtres, cellules, iode). |
| `/radhelp` | Aide radiation (indispensable dès le tier 4+). |
| `/inscribe` | Graver un scroll EcoScrolls sur un objet (pas d'enclume/table d'ench.). |
| `/forgeron` | Atelier de réparation d'équipement. |
| `/trade` | Échange entre joueurs (module AxTrade absorbé). |

### Joueur — reports / feedback (WebhookReport)

| Commande | Effet |
| --- | --- |
| `/reportplayer <pseudo> <commentaire>` | Signaler un joueur. |
| `/reportbug <commentaire>` | Signaler un bug (ouvre un fil Discord). |
| `/avis <commentaire>` | Laisser un avis 1–5. |
| `/suggestion <texte>` | Suggestion (réactions 👍/👎 + fil auto). |

### Admin — SurvivorCore & modules

| Commande | Effet |
| --- | --- |
| `/sc status` / `/sc reload` | État des modules + créneau actif / recharge. |
| `/sc slot list\|info\|force\|stop\|next` | Gérer les créneaux (Nuit Creuse, Rush Weekend…). |
| `/radiation reload`, `/radzone`, `/radkit` | Admin radiation. |
| `/discordlink status\|info\|unlink\|reload\|queue` | Admin liaison Discord. |
| `/svnarrate [--broadcast] <texte>` | Narration IA d'un fait (Discord ± in-game). |
| `/svnarrate-bilan` | Bilan quotidien (console, schedule 22h). |
| `/svflavor <cible> <mode> <texte>` | Reformulation IA affichée en jeu. |
| `/webhook history\|search\|delete\|note\|blacklist\|stats\|top\|admin reload` | Admin reports. |

### Bot Discord (slash)

`/embed …` (créer/poster des embeds), commandes du panel pour autoreact / autorole / mc-announce / welcome / tickets.

---

## 4. Pièges / gotchas connus (utiles pour créer du contenu)

- **Placeholders non remplacés** : dans les annonces `mc.announce` et `welcome`, un `%clef%` sans valeur reste **affiché tel quel**. Toujours fournir toutes les variables qu'un template attend. Format welcome : `%member_mention%`, `%member_count_ordinal%`, `%rules_channel%`, etc.
- **Couleur d'embed = ton** : ne jamais mettre une alerte danger en vert (voir doc 06, charte émotion→couleur).
- **Trois "tiers" distincts** : zone (danger 0→5), prime (économie, label coloré), grade survivant (rankup). Ne pas les mélanger dans le contenu (voir doc 07 §6).
- **Radiation = mort lente**, jamais instantanée. Pas de « tu meurs sur le coup » à 8000 mSv.
- **Pas de table d'enchantement / enclume** : la puissance vient des scrolls (`/inscribe`).
- **Slugs de zones ≠ noms affichés** : `qg_police` (slug WG) s'affiche « QG de Police ». Pour le matching technique on garde le slug ; pour le contenu lisible on utilise le nom joli.
- **Anomalies STALKER & spawn de mutants par signature** : mécaniquement **retirés** (2026-05-18). Toujours valables comme **ambiance/lore**, mais ne pas promettre une mécanique active.
- **Pas d'emoji dans la prose immersive** (narration / flavor) — emojis réservés aux titres et fields d'annonce.
- **Pas de message de départ** : le module welcome ne poste rien quand un membre quitte.
- **Le bot est le seul à parler à l'API Discord** (rate-limits) : les opérations post-webhook (réactions, fils) passent par lui (`webhookbridge`).
- **Footer signature** : `MaSurvie · masurvie.fr` (le domaine sert d'adresse de connexion). Footer du narrateur = nom de la persona.
- **Pseudo MC toujours affiché** dans les reports/embeds ; le Discord est un complément (« (non lié) » si pas lié).

---

## 5. Récapitulatif des secrets à NE JAMAIS divulguer

| Secret | Où il vit |
| --- | --- |
| Token du bot Discord, clés API Anthropic, tokens partagés | `bot/.env` (non divulgué) |
| URLs de webhooks Discord | `plugins/WebhookReport/webhooks.yml` (non divulgué) |
| Identifiants / mot de passe MySQL | `plugins/SurvivorCore/bdd.yml`, `bot/.env`, `survivor-panel/.env` (non divulgués) |
| OAuth Discord, clés session/chiffrement, clé API Pterodactyl | `survivor-panel/.env` (non divulgué) |

Si l'un de ces éléments apparaît, mentionner uniquement « configuré dans \<fichier\> (non divulgué) ».

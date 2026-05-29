# 06 — Style des annonces Discord (MaSurvie)

Ce document décrit **comment écrire une annonce / un embed Discord dans le style maison** du serveur MaSurvie (Skyblock + survie post-apo zombie, `masurvie.fr`). Il est destiné à une IA qui doit produire des annonces (events, maintenances, nouveautés, lore) directement réutilisables.

> **Aucun secret ici.** Les tokens de bot, URLs de webhooks, identifiants MySQL et clés API sont stockés dans `bot/.env`, `webhooks.yml` et `bdd.yml` (non divulgués). On ne les copie jamais.

---

## 1. Comment les annonces sont produites (techniquement)

Le serveur a **un bot Discord maison** (dossier `bot/`, Node.js + discord.js) et un **panneau d'admin web** (`survivor-panel/`). Les annonces partent de l'un de ces trois canaux :

1. **Embeds composés à la main** — module bot `embeds` + éditeur visuel "Draftbot-like" du panel (`/embeds`). On y règle titre, URL, bandeau auteur, description (markdown), couleur, image, miniature, jusqu'à 25 fields, footer, timestamp, et des boutons / menus déroulants. C'est le canal pour les **annonces rédigées** (events, règlements, changelogs, nouveautés).
2. **Annonces déclenchées depuis le jeu** (`mc.announce`) — le plugin Minecraft pousse un event WebSocket `mc.announce` ; le bot charge un **template** stocké en base (`mb_mc_announce_templates`), remplace les `%placeholders%` et poste l'embed. C'est le canal **automatique** (un boss tombe, un créneau démarre, un palier serveur atteint…).
3. **Narrateur IA** (modules `narrator` côté bot + `flavortext` en jeu) — un fait brut est transformé par Claude en message d'ambiance immersif (voir §6).

### Format d'un embed (champs disponibles)

Le builder du bot (`bot/src/modules/embeds/render.js`) accepte le payload JSON suivant :

```jsonc
{
  "title": "…",          // 256 car. max
  "url": "https://…",     // titre cliquable
  "description": "…",     // 4096 car. max, markdown Discord
  "color": "#5865F2",     // hex (#RRGGBB) OU entier
  "author": { "name": "…", "icon_url": "…", "url": "…" },
  "footer": { "text": "…", "icon_url": "…" },  // 2048 car.
  "thumbnail": "https://…",  // petite image en haut à droite
  "image": "https://…",      // grande image / bannière
  "timestamp": true,         // true = maintenant, ou une date ISO
  "fields": [ { "name": "…", "value": "…", "inline": true } ]  // 25 max
}
```

Composants interactifs : 5 lignes max ; **une ligne = 5 boutons OU 1 menu déroulant** (jamais mélangés). Styles de bouton : `primary` (blurple), `secondary` (gris), `success` (vert), `danger` (rouge), `link` (vers une URL).

---

## 2. La palette de couleurs maison

Les couleurs réellement utilisées dans le code/bot, à réutiliser pour rester cohérent :

| Usage | Couleur | Hex / int |
| --- | --- | --- |
| **Embed par défaut** (commande `/embed create` sans couleur) | Blurple Discord | `#5865F2` |
| **Narration ponctuelle par défaut** (narrateur, ambiance neutre) | Gris-bleu | `#607D8B` |
| **Bilan quotidien 22 h** ("Chroniques du jour") | Gris ardoise sombre | `#2C2F33` (= `0x2C2F33`) |

### Mapping émotion → couleur (charte du narrateur, à suivre pour TOUTE annonce d'event)

C'est la `color-guidance` du lore (`narrator.yml`). Choisis la couleur de l'embed selon le **ton** de l'annonce :

| Ton de l'annonce | Couleur |
| --- | --- |
| Exploit héroïque / boss tombé / réussite éclatante | doré / orange |
| Mort tragique / trahison / mauvaise nouvelle | rouge sombre |
| Découverte / butin rare / nouveauté précieuse | violet |
| Routine / petit gain / info de fond | gris-bleu (`#607D8B`) |
| Événement collectif positif (event communautaire, capture de territoire) | vert |

Suggestions d'hex cohérentes : doré `#FACC15`, orange `#E67E22`, rouge sombre `#992D22` / `#C0392B`, violet `#9B59B6`, vert `#2ECC71`, gris-bleu `#607D8B`.

---

## 3. Structure d'une annonce maison

Ordre recommandé (descendant) :

1. **Titre** court et thématique, souvent avec **un emoji de tête** qui code la catégorie (voir §4). Ex. `☢ ALERTE — Nuit Creuse en cours`.
2. **Description** : 1 à 3 phrases d'accroche immersives (registre post-apo, voir doc 07), puis l'info utile. Markdown autorisé : **gras** pour les mots-clés, `>` pour une citation d'ambiance, listes à puces pour les détails.
3. **Fields** (optionnels) pour les infos structurées : `Quand`, `Où`, `Récompense`, `Durée`, `Comment participer`. Mettre `inline: true` pour les aligner sur une ligne.
4. **Image / bannière** pour les gros events ; **miniature** pour un visuel d'item ou d'icône.
5. **Footer** : signature serveur + IP. Footer recommandé : `MaSurvie · masurvie.fr` (le domaine fait office d'IP de connexion). Pour le narrateur, le footer = **le nom de la persona** (ex. `Radio Survivants`).
6. **Timestamp** : activé sur les annonces datées (events, maintenances).
7. **Boutons** : `link` vers le site / le règlement, ou `primary` pour une action gérée par le bot.

### Mentions / rôles

- Pour toucher tout le monde : `@everyone` (réservé aux gros events / maintenances). Pour un public ciblé : mention d'un **rôle** (`<@&ROLE_ID>`). Les IDs de rôles ne sont pas en dur dans le repo public.
- Le **propriétaire** du panel a un ID Discord défini côté privé (non divulgué dans ce repo public) — ne jamais inventer ni divulguer d'IDs Discord.

---

## 4. Emojis récurrents (vocabulaire visuel)

Repris du wiki et du lore. Un emoji de tête de titre code la catégorie :

| Emoji | Sens |
| --- | --- |
| ☢ | Radiation, danger, alerte, zones profondes |
| 🧟 / 🦠 | Zombies, infectés, mutants, hordes, convois |
| 🗺️ | Zones, carte, territoires |
| 📦 | Loot, coffres, butin |
| 🎯 | Event, objectif, convoi |
| ⏰ / ⚡ | Créneau dynamique (Nuit Creuse, Rush Weekend) |
| 🏰 | Factions, territoires |
| ⭐ / 🎖️ | Rang, palier, exploit, top |
| 💰 / 🪙 | Économie, prime, crédits, récompense |
| 🛠️ / 🔧 | Maintenance, mise à jour, redémarrage |
| 🛡️ | Protection, équipement, sécurité |
| 📡 / 🎙️ | Radio, annonce, communiqué |
| 🎉 ✅ ❌ 👍 👎 | Réactions/validations (autoreact, suggestions) |

> Note : **dans le corps d'une narration immersive (texte du narrateur), pas d'emoji** — le style l'interdit. Les emojis vont dans les titres, fields et annonces "officielles", pas dans la prose d'ambiance.

---

## 5. Exemple complet — annonce d'event "Nuit Creuse"

Payload (style maison reproductible) :

```jsonc
{
  "title": "☢ ALERTE — La Nuit Creuse est tombée",
  "color": "#992D22",
  "description": "> *« Le soleil s'est couché sur les ruines. Les Geiger crépitent plus fort.\n> Ils sont plus nombreux cette nuit. Restez groupés, survivants. »*\n\nLa **Nuit Creuse** est active : le **loot** des zones est boosté, mais le **danger** aussi. Les hordes grossissent, les zones profondes irradient davantage. C'est le moment de piller gros — ou de mourir bête.",
  "fields": [
    { "name": "⏰ Durée", "value": "Toute la nuit en jeu", "inline": true },
    { "name": "📦 Bonus", "value": "Loot & drop renforcés", "inline": true },
    { "name": "🧟 Risque", "value": "Hordes + radiation accrues", "inline": true },
    { "name": "🛡️ Conseil", "value": "Filtres, iode et trousse de soins **avant** de sortir. Décontaminez au retour." }
  ],
  "image": "https://…/banniere_nuit_creuse.png",
  "footer": { "text": "MaSurvie · masurvie.fr" },
  "timestamp": true
}
```

### Variante "maintenance" (ton sobre, gris-bleu)

```jsonc
{
  "title": "🛠️ Maintenance programmée",
  "color": "#607D8B",
  "description": "Le serveur redémarre **ce soir à 22h00** pour une mise à jour. Coupure estimée : ~15 min. Mettez votre butin à l'abri avant la fermeture.",
  "fields": [
    { "name": "🕘 Début", "value": "22:00 (heure de Paris)", "inline": true },
    { "name": "⏳ Durée", "value": "~15 minutes", "inline": true }
  ],
  "footer": { "text": "MaSurvie · masurvie.fr" },
  "timestamp": true
}
```

### Variante "nouveauté" (ton positif/découverte, violet)

```jsonc
{
  "title": "📦 Nouveauté — Le Forgeron rouvre boutique",
  "color": "#9B59B6",
  "description": "Un nouvel atelier de **réparation** est disponible au spawn. Réparez vos pioches du pilleur et vos combis NBC abîmées sans perdre leurs inscriptions.\n\nTapez **/forgeron** en jeu pour commencer.",
  "footer": { "text": "MaSurvie · masurvie.fr" }
}
```

---

## 6. Déclencheurs automatiques (à connaître pour le ton)

### a) Message de bienvenue (module `welcome`)

À l'arrivée d'un membre (`guildMemberAdd`), le bot poste un embed public **et/ou** envoie un DM (échec silencieux si DM fermés). Placeholders disponibles dans le texte :

- `%member_mention%` (mention `<@id>`), `%member_name%`, `%member_tag%`, `%member_id%`
- `%member_count%` (nombre de membres), `%member_count_ordinal%` (ex. « 1250e membre »)
- `%guild_name%`, `%rules_channel%` (lien vers le salon règlement)

Avatar du membre utilisable en miniature et/ou grande image. Pas de message de départ. Ton attendu : accueil chaleureux teinté de lore (« Bienvenue parmi les survivants, %member_name% »).

### b) Annonces MC → Discord (`mc.announce`)

Le plugin pousse `{ template_key, channel_id?, placeholders: {...} }`. Le bot remplit le template stocké en base. Les placeholders sont au format `%clef%` et sont remplacés partout (titre, description, fields, footer). Si un placeholder n'a pas de valeur, il est **laissé tel quel** (`%clef%`) — donc toujours fournir les variables attendues par le template. Endpoint de test côté panel : `POST /api/internal/mc-announce/test`.

### c) Réactions automatiques (`autoreact`)

Sur certains salons (configurés en base `mb_autoreact_rules`), le bot ajoute automatiquement une liste d'emojis à **chaque message** (max 20 emojis dédupliqués, throttle ~120 ms). Usage typique : un salon "suggestions" où chaque post reçoit 👍 / 👎 automatiquement. (Les suggestions in-game via `/suggestion` reçoivent aussi 👍/👎 + un fil, via le module `webhookbridge`.)

### d) Auto-rôles (`autorole`)

5 déclencheurs : `on_join`, `button`, `reaction` (reaction-roles), `condition` (boost, compte lié, présence dans une région, palier de prime), `tenure` (après N minutes de présence). Utile à savoir pour rédiger les **embeds de reaction-roles** (ex. « Réagis avec 🗺️ pour suivre les annonces de zones »).

---

## 7. Règles d'or de rédaction

- **Toujours en français.** Registre immersif post-apo dans la prose (voir doc 07), info pratique claire dans les fields.
- **Couleur = ton** (charte §2). Une alerte danger n'est jamais en vert.
- **Footer = signature serveur** (`MaSurvie · masurvie.fr`) sauf narration (footer = persona).
- **Emojis dans titres/fields, pas dans la prose immersive.**
- **Concret > vague** : noms de zones réels, montants, durées, commandes en `/code`.
- **Jamais de secret** : pas de token, pas d'URL de webhook, pas d'identifiant MySQL.

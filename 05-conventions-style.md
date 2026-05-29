# 05 — Conventions de style (DA MaSurvie V2) — FICHIER LE PLUS IMPORTANT

> Cerveau de contexte public. Ce fichier est **la bible visuelle** du serveur. Toute IA
> qui crée un item, un menu, un succès, un lore ou une annonce DOIT en sortir des
> couleurs, raretés, glyphes et formats **identiques** à ceux décrits ici. Codes hex
> exacts, pas d'approximation.

La **source de vérité absolue** est le bloc `palette:` + `prefixes:` en tête de
`plugins/SurvivorCore/lang_fr.yml` (alias dépôt :
`masurvie_mini_core/src/main/resources/lang_fr.yml`, ~106 KB). Tout y est centralisé :
changer la DA = éditer la palette + reload, sans toucher les messages.

---

## 1 · Palette de couleurs OFFICIELLE (hex exacts)

DA post-apocalyptique : **Sang vif / Sang profond / Parchemin / Vert toxique**.
Ces tokens sont substitués par le `LangResolver` AVANT le rendu MiniMessage.

### Identité principale
| Token | Hex | Rôle |
|---|---|---|
| `<primary>` | `#DC2626` | Sang principal — boutons, accents, titres |
| `<secondary>` | `#7F1D1D` | Sang profond — fin de gradient, sombre |
| `<accent>` | `#EF4444` | Sang vif — hover, états alerte |

### Textes
| Token | Hex | Rôle |
|---|---|---|
| `<bright>` | `#ECEDEE` | Texte clair sur fond sombre |
| `<text>` | `#B5B5B8` | Texte secondaire (descriptions) |
| `<dark>` | `#6B6B70` | Texte discret (labels, metadata) |
| `<darkest>` | `#2A2A36` | Bordures, séparateurs |

### Sémantique
| Token | Hex | Rôle |
|---|---|---|
| `<success>` | `#84CC16` | Vert toxique — online, OK, positif |
| `<warning>` | `#F59E0B` | Avertissements, jauges |
| `<parchment>` | `#D4C5A0` | Os / parchemin — top, accents secondaires |
| `<paper>` | `#E8D9B4` | Papier vieilli — sous-titres post-apo |

### Fonds (référence visuelle, peu utilisés en chat)
| Token | Hex |
|---|---|
| `<background>` | `#0A0A0B` |
| `<panel>` | `#14141A` |

### Gradient identité serveur
Le nom du serveur s'écrit toujours en **gradient sang** :
```
<gradient:#DC2626:#7F1D1D><bold>MaSurvie</bold></gradient>
```

---

## 2 · Préfixes réutilisables (`<prefix:xxx>`)

Séparateur vertical `┃` après un mot-clé en gras + couleur sémantique :

| Token | Rendu (hex inclus) |
|---|---|
| `<prefix:main>` | `<gradient:#DC2626:#7F1D1D><bold>MaSurvie</bold></gradient> <#B5B5B8>• ` |
| `<prefix:error>` | `<#EF4444><bold>ERREUR</bold></#EF4444> <#B5B5B8>┃ ` |
| `<prefix:success>` | `<#84CC16><bold>SUCCÈS</bold></#84CC16> <#B5B5B8>┃ ` |
| `<prefix:warning>` | `<#F59E0B><bold>ATTENTION</bold></#F59E0B> <#B5B5B8>┃ ` |
| `<prefix:info>` | `<#D4C5A0><bold>INFO</bold></#D4C5A0> <#B5B5B8>┃ ` |
| `<prefix:server-name>` | gradient MaSurvie seul (sans suffixe) |

Séparateurs décoratifs :
- `<sep-line>` = `<#2A2A36>▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪▪</#2A2A36>` (long)
- `<sep-short>` = `<#2A2A36>▪▪▪▪▪▪▪▪▪▪▪▪▪</#2A2A36>` (court)

Le préfixe global par défaut (`prefix:` dans lang_fr.yml) = `<prefix:main>`.

---

## 3 · Raretés — Commun → Divin (5 niveaux + étoiles)

Le serveur a **5 raretés**, alignées sur les **5 tiers d'étoiles** (1★→5★). C'est
l'enum officielle des moteurs d'items : **`COMMUN` / `RARE` / `EPIQUE` / `LEGENDAIRE`
/ `DIVIN`** (MaSurvieOutil `rarity:` ; majuscules, sans accent dans l'enum).

> ⚠️ **Deux conventions de couleur de rareté coexistent dans le dépôt.** Privilégier
> les **étoiles `<starN>`** (DA officielle 2026-05-27) pour tout nouveau contenu, et
> garder en tête la palette « loot » alternative pour cohérence avec l'existant.

### A. Convention officielle DA — étoiles `<star1>`→`<star5>`
C'est la convention à utiliser **en priorité**. Chaque rareté = un palier d'étoiles
en gras.

| Rareté | Étoiles | Token | Hex | Glyphe |
|---|---|---|---|---|
| **Commun** (1★) | ★ | `<star1>` | `#B5B5B8` | `<#B5B5B8><bold>★</bold></#B5B5B8>` |
| **Rare** (2★) | ★★ | `<star2>` | `#D4C5A0` | `<#D4C5A0><bold>★★</bold></#D4C5A0>` |
| **Épique** (3★) | ★★★ | `<star3>` | `#F59E0B` | `<#F59E0B><bold>★★★</bold></#F59E0B>` |
| **Légendaire** (4★) | ★★★★ | `<star4>` | `#EF4444` | `<#EF4444><bold>★★★★</bold></#EF4444>` |
| **Divin** (5★) | ★★★★★ | `<star5>` | `#7F1D1D` | `<#7F1D1D><bold>★★★★★</bold></#7F1D1D>` |

### B. Palette « loot » alternative (tokens nommés)
Codée dans `palette:` sous le commentaire « alt (loot …) ». Utilisée par certains
modules (LootRegion, EpicAchievements, killstreaks) pour colorer le **nom** d'une
rareté ou d'un drop.

| Rareté | Token | Hex |
|---|---|---|
| Commun | `<commun>` | `#929292` |
| Rare | `<rare>` | `#9FFFE9` (cyan clair) |
| Épique | `<epique>` | `#D26EE3` (violet/magenta) |
| Légendaire | `<legendaire>` | `#FF5B4E` (rouge-corail) |
| Divin | `<divin>` | `#D29E27` (or) |

### C. Variante « tiers » RegionStats (encore une autre, pour info)
Vue dans `lang_fr.yml` (section drops/loot) — à NE PAS généraliser, mais ne pas s'en
étonner :
`tier_common: <gray>Commun` · `tier_uncommon: <success>Peu commun` ·
`tier_rare: <#2980B9>Rare` · `tier_epic: <epique>Epique` ·
`tier_legendary: <parchment>Legendaire`.

> Règle pratique pour créer un item : choisir l'enum (`COMMUN`…`DIVIN`), et colorer le
> nom soit avec l'étoile `<starN>` (DA officielle), soit avec le token loot
> `<commun/rare/epique/legendaire/divin>` selon le contexte du module ciblé.

---

## 4 · ItemsAdder — namespaces, model_data, glyphes

### Namespaces
| Namespace | Usage |
|---|---|
| `masurvie` | Namespace **principal** des items custom IA (ex. `masurvie:pied_de_biche`). Fichiers : `plugins/ItemsAdder/contents/masurvie/items/<id>.yml`, bloc `info: { namespace: masurvie }`. |
| `infected` | Namespace historique utilisé dans le loot LootRegion (ex. `infected:noyau_du_labo`). |

Format de référence d'un item IA (réel) :
```yaml
info:
  namespace: masurvie
items:
  pied_de_biche:
    display_name: "&6Pied-de-biche du Labo"
    lore:
      - "&7Outil de fouille du Labo."
      - "&8Réparable à la Forge MaSurvie."
    resource:
      material: IRON_PICKAXE
      generate: false           # réutilise le modèle vanilla → pas de RP à régénérer
    durability:
      max: 250                  # durabilité custom IA (réparée par la Forge)
      vanilla_durability_bar: true
```
Les **trophées** (TrophyDisplay) sont aussi des items IA uniques par joueur (soulbound),
posables dans le monde avec hologramme FancyHolograms.

### Plages de custom_model_data (conventions observées)
Issues du moteur **MaSurvieOutil** (le plus précis sur les CMD) :

| Plage | Catégorie | Exemples réels |
|---|---|---|
| `1000x` (≈ 10000–10999) | **Items** (consommables, outils, armes, gadgets) | `vodka_du_stalker` = `10006` |
| `2000x` (≈ 20000–20999) | **Scrolls / parchemins** | `chasseur_de_mutants` = `20013` |

> ⚠️ Ce sont les plages **constatées** dans le dépôt (pas un registre formel écrit). Pour
> créer un nouvel item : viser `100xx` ; un nouveau scroll : `200xx`. **Confirmer**
> qu'il n'y a pas de collision avec le RP existant côté serveur.

### Glyphes / font_images
> ⚠️ À CONFIRMER : **aucun fichier `font_images:` / `glyph:` ItemsAdder** n'est présent
> dans le dépôt. Les « glyphes » utilisés dans le contenu sont en réalité des **caractères
> Unicode** dans les noms/lore (voir §6), pas des images de police IA. Si un atlas de
> glyphes existe, il vit côté serveur hors dépôt.

---

## 5 · Deux dialectes de couleur : MiniMessage vs legacy `&`

Le serveur mélange **deux formats** selon le sous-système — important à respecter :

- **MiniMessage + tokens DA** (`<primary>`, `<accent>`, `<#RRGGBB>`, `<gradient:…>`,
  `<bold>`) : c'est le **format des messages** (chat, titles, broadcasts, GUI récents,
  lore MaSurvieOutil). **À utiliser pour tout nouveau message / lore moderne.**
- **Codes legacy `&`** (`&6`, `&7`, `&a`, `&c`, `&l`, et hex legacy `&#RRGGBB` ou
  `§x`) : encore utilisés dans **ItemModule** (`itemmodule.yml`), les items IA, et
  quelques configs anciennes. Respecter le format **du fichier que tu édites**.

Hex legacy vus : `&#1E2A35` (titres StreamCode), `&#33FF33` (logs console Trade).

---

## 6 · Conventions de nommage & format des items

### IDs d'items / scrolls
- **snake_case**, sans accent, descriptif : `serum_anti_radiation`, `combinaison_nbc_legere`,
  `cartouche_filtre_carbone`, `capsule_brouilleur`, `chasseur_de_mutants`.
- **Suffixe de tier** : `_t2`, `_t3`, `_t4` (ex. `pioche_pilleur_t2`, `pickaxe_pilleur_t4`)
  ou un libellé final : `_legendaire` (`pickaxe_pilleur_legendaire`).
- Préfixe `im:` pour **référencer un item ItemModule** dans un bouton d'inventaire
  (ex. `material: "im:potion_foret"`).

### Glyphes Unicode récurrents dans noms/lore
Le contenu utilise des **symboles Unicode** dans les noms d'items pour l'ambiance :
- `⛑` masque/casque · `☣` biohazard (combi NBC) · `☢` radiation (combi lourde) ·
  `⚙` engrenage (exosquelette) · `⚗` alambic (cartouche filtre) · `⚡` éclair (cellule) ·
  `🧵` couture (kit textile) · `⚛` atome (protection iode) · `▸`/`▶`/`▪` puces de lore ·
  `★` étoiles de rareté · `⚠` avertissement · `✔` validation · `🧬` mutation · `🩸`/`💊`.

### Format de lore d'un item (modèle réel à imiter)
Structure type d'un item d'équipement (ici legacy `&`, format ItemModule) :
```yaml
name: "&7&l⛑ Masque filtrant"
lore:
  - "&8&oMasque équipé d'un filtre HEPA qui retient"   # description italique grise
  - "&8&oles particules radioactives en suspension."
  - ""                                                  # ligne vide = séparateur
  - "&7▸ Slot : &fCasque"                               # attributs : "&7▸ <label> : &f<valeur>"
  - "&7▸ Protection : &b15%&7 &8(si filtre actif)"
  - ""
  - "&6▶ État du filtre"                                # sous-titre orange "&6▶"
  - "&7  Charges : &e%filter%&7/&e%filter_max% &8(&e%filter_percent%&8)"
  - ""
  - "&c⚠ Sans charges, protection nulle."               # avertissement rouge "&c⚠"
  - "&8À porter sur la tête."                           # note d'usage grise
```

Conventions de lore observées :
- **Description** : `&8&o…` (gris foncé italique), 1–2 lignes en tête.
- **Ligne vide `""`** pour aérer les sections.
- **Attribut** : `&7▸ <Label> : &f<valeur>` (puce `▸`, label gris, valeur blanche/colorée).
- **Sous-titre de bloc** : `&6▶ <Titre>` (orange).
- **Avertissement** : `&c⚠ …` (rouge).
- **Note d'usage / mode d'emploi** : `&8 …` (gris discret).
- **Placeholders d'usure** dans le lore : `%durability%`, `%max_durability%`,
  `%filter%`, `%filter_max%`, `%filter_percent%`, `%integrity%`, `%integrity_percent%`,
  `%exo_status%`, `%exo_minutes%` (cf. fichier ItemModule).

Modèle de lore **moderne** (MiniMessage, MaSurvieOutil) :
```yaml
display_name: "<#aaaaaa>Vodka du Stalker</#aaaaaa>"
rarity: COMMUN
lore:
  - "<#aaaaaa>Tradition incontournable de la Zone."
  - ""
  - "<#f8a5c2>Effet</#f8a5c2><white>: -500 mSv</white>"
```
Pour un scroll : nom en gradient (`<gradient:#c9184a:#ff4d6d>Parchemin : Chasseur de
Mutants</gradient>`) + un `effect_summary:` court coloré.

---

## 7 · Conventions de placeholders

- **Préfixe = nom du plugin/module en minuscule**, encadré de `%…%` :
  `%survivorcore_*%`, `%lootregion_*%`, `%regionstats_*%`, `%radiationcore_*%`,
  `%territory_*%`, `%eventhub_*%`, `%zonespawn_*%`, `%trophe_*%`, `%webhook_*%`,
  `%horde_*%`, `%streamcode_*%`, `%reacttracker_*%`, `%infectedconvoy_*%`.
- **snake_case** dans le nom du placeholder ; arguments variables collés en suffixe
  (`%lootregion_chests_opened_<region>%`, `%eventhub_counter_<TYPE>%`).
- **Nombres = entiers** quand consommés par EpicAchievements (EA rejette les décimaux —
  d'où `%survivorcore_balance%` arrondi `Math.floor`, et l'usage de
  `%vault_eco_balance_fixed%` au lieu de `%vault_eco_balance%`).
- **Placeholders internes (résolus par le plugin avant envoi)** : `%victim%`, `%killer%`,
  `%mob%`, `%amount%`, `%player%`, `%name%`, `%remaining%`, `%count%`… (format
  `%mot%`, idem PAPI mais résolus en amont).
- Le **lang central** documente les deux familles en en-tête.

---

## 8 · Exemples concrets copiables (vrais codes DA)

**Message d'erreur** :
```
<prefix:error>Tu n'as pas la permission d'entrer dans cette zone.
```
(rend : `ERREUR` rouge `#EF4444` gras + `┃` + texte `#B5B5B8`)

**Message de succès** :
```
<prefix:success>Coffre pillé — <gold>+250</gold> <text>capsules.
```

**Ligne de menu / item de rareté épique (étoiles DA)** :
```
<star3> <epique>Épée du Chasseur</epique> <text>— +25% dégâts vs contaminés
```

**Lore type (item moderne MiniMessage)** :
```yaml
display_name: "<legendaire>Pioche du Pilleur</legendaire>"
lore:
  - "<text>Fouille quasi instantanée des coffres."
  - ""
  - "<warning>▸</warning> <text>Bonus drop : <success>+20%</success>"
  - "<star4>"
  - "<dark>Trouvée dans les coffres tier 4-5."
```

**Annonce killstreak (réel, palette loot)** :
```
<epique>%killer% est en feu ! 5 kills !</epique>
<gradient:#FFAA00:#FF5555>%killer% EST UN MONSTRE - 10 KILLS !!</gradient>
```

**Titre de section / séparateur** :
```
<sep-line>
<prefix:info>Statistiques de la zone
<sep-line>
```

---

## Valeurs NON trouvées (à confirmer par l'admin)

> ⚠️ Pas de registre formel des plages `custom_model_data` (uniquement des exemples
> `100xx` items / `200xx` scrolls). À figer pour éviter les collisions RP.
>
> ⚠️ Pas de fichier `font_images:` / `glyph:` ItemsAdder dans le dépôt — les « glyphes »
> sont des caractères Unicode. Confirmer s'il existe un atlas de police custom serveur.
>
> ⚠️ Trois conventions de couleur de rareté coexistent (étoiles DA / tokens « loot » /
> tiers RegionStats). L'admin devrait trancher **une** convention canonique pour les
> nouveaux items si l'uniformité visuelle est souhaitée.

---

## Sources lues pour ce fichier

- `masurvie_mini_core/src/main/resources/lang_fr.yml` — blocs `palette:` + `prefixes:` (hex exacts, étoiles, séparateurs)
- `deploy/configs/SurvivorCore/lang_fr.yml` — tiers de rareté RegionStats, hex legacy
- `deploy/configs/SurvivorCore/itemmodule.yml` — lore type, glyphes, format `&` legacy, naming
- `wiki/itemmodule/README.md` — placeholders d'usure, IDs, préfixe `im:`
- `wiki/masurvieoutil/README.md` + `ARCHITECTURE.md` — enum rareté `COMMUN…DIVIN`, `custom_model_data` 10006 / 20013, lore MiniMessage
- `wiki/eco_stack/ecoitems_masurvie.md` + `ecoscrolls_masurvie.md` — naming `_t2`/`_legendaire`, raretés scrolls
- `deploy/server-content/ItemsAdder/contents/masurvie/items/pied_de_biche.yml` — namespace `masurvie`, format item IA
- `wiki/placeholders.md` — préfixes et conventions de placeholders
- `wiki/lootregion/README.md` — namespaces `masurvie` / `infected`

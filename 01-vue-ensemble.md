# 01 — Vue d'ensemble du serveur MaSurvie V2

> Cerveau de contexte public. Ce fichier décrit **l'identité**, **la philosophie** et
> **les boucles de jeu** du serveur. Toute IA qui veut créer du contenu (configs,
> succès, lore, annonces, menus) doit lire ce fichier en premier pour rester dans le
> bon ton et le bon univers.

---

## 1 · Identité

| Champ | Valeur |
|---|---|
| **Nom** | MaSurvie V2 |
| **Nature** | Serveur Minecraft **Skyblock** (moteur **SuperiorSkyblock2**) avec un **overlay de survie post-apocalyptique zombie** par-dessus. |
| **Domaine / site web** | `masurvie.fr` (déclaré dans le `plugin.yml` de SurvivorCore : `website: https://masurvie.fr`). |
| **Plugin socle** | **SurvivorCore** — plugin modulaire maison qui a absorbé ~22-24 anciens plugins en modules (cf. fichier 02). |
| **Logiciel serveur** | **Purpur 1.21.x** (basé sur Paper). Le code maison cible `api-version: '1.21'`, les `pom.xml` pointent `purpur-api 1.21.4` et `paper-api 1.21.1 / 1.21.8`. Plusieurs commentaires de code mentionnent explicitement « MaSurvie est sur Purpur » et des quirks « Purpur 1.21.11 ». |
| **Version Java** | **Java 21** (toutes les sources : `sourceCompatibility 21`, `<java.version>21</java.version>`). |
| **Hébergeur** | **Minestrator** (panel **Pterodactyl**). La base MySQL est infogérée chez l'hébergeur ; le bot/panel tourne sur un VPS distinct. _(Hôtes/identifiants exacts = privés, non divulgués ici.)_ |
| **Base de données** | MySQL partagée (chez l'hébergeur) avec fallback SQLite par module ; driver `mysql-connector-j 8.4.0`, pool HikariCP. |

### Valeurs d'identité NON trouvées dans le dépôt (à confirmer par l'admin)

> ⚠️ À CONFIRMER PAR L'ADMIN : **l'IP / adresse de connexion en jeu** (aucun
> `server.properties`, MOTD ni adresse `play.masurvie.fr` n'existe dans le dépôt).

> ⚠️ À CONFIRMER PAR L'ADMIN : **la build Purpur exacte** (la famille 1.21.x est
> certaine ; des commentaires de code citent 1.21.1, 1.21.4, 1.21.8 et 1.21.11
> selon les modules, mais le numéro de build qui tourne réellement n'est pas figé
> dans le dépôt).

> ⚠️ À CONFIRMER PAR L'ADMIN : **le MOTD et le footer d'annonces** (aucun trouvé).

---

## 2 · Philosophie du serveur

### Le pitch

**MaSurvie** est un serveur de **survie post-apocalyptique**. Le joueur est un
**survivant** lâché dans un monde **contaminé par la radiation**. Il **pille des
zones dangereuses** pour s'équiper, **conquiert des territoires** avec sa faction, et
surtout **survit à la radiation** qui le ronge à chaque expédition profonde. Le
principe central : **plus on va loin, plus le loot est bon — et plus le monde veut
sa peau.**

Techniquement, le socle Skyblock vient de **SuperiorSkyblock2** (îles, équipes), mais
le cœur du gameplay est l'overlay survie : zones de danger, radiation, mutants,
hordes, territoires et convois.

### Les piliers (univers et ton)

1. **🗺️ Les ~20 zones (tier 0 → 5)** — la carte est découpée en régions WorldGuard
   classées par tier de danger : tier 0 = spawn sûr, tier 5 = zone létale à
   artefacts. Plus le tier monte : mobs plus forts, radiation plus élevée, loot plus
   rare.
2. **📦 Les coffres LootRegion** — coffres pillables par zone, avec cooldown,
   animation de fouille, pity, streaks. Le contenu dépend du tier de la zone et de
   l'équipement du joueur.
3. **🎯 Items & scrolls** — objets puissants custom (anciennement EcoItems, désormais
   le moteur **ItemModule**/MaSurvieOutil) et parchemins gravables via `/inscribe`.
   **Pas de table d'enchantement vanilla** : la puissance vient des scrolls.
4. **☢ Système de radiation** — la dose (en mSv) monte en zone profonde ; sans
   protection (masque + filtre, combi NBC, exosquelette) elle devient létale (mort
   lente à 8000 mSv). Décontamination, iode et filtres sont vitaux.
5. **🦠 Les mutants** — rester contaminé augmente la **signature**, qui attire des
   créatures mutantes (Bloodsucker, Chimera…) qui traquent le joueur sale.
6. **⚡ Anomalies STALKER** — pièges invisibles dans les zones radioactives, détectés
   au boulon, mortels, qui abritent les artefacts les plus précieux.
7. **🏰 Territoires** — les factions capturent des zones WorldGuard : bonus passifs,
   taxe sur les pilleurs, zones contestables.
8. **⏰ Créneaux dynamiques** — fenêtres de temps qui changent les règles (Nuit
   Creuse = loot/danger boostés la nuit ; Rush Weekend = gains accrus le week-end).
   Certains effets d'objets ne s'activent que « pendant un créneau ».
9. **🎯 Convois zombies** — convois d'infectés qui traversent la map à intervalles ;
   les intercepter = loot d'événement, mais ils sont coriaces et mobiles.
10. **🧟 Hordes dynamiques** — illusion d'une horde infinie : où qu'aille le joueur,
    des silhouettes émergent, en nombre adapté au danger de la zone (recyclage des
    mobs lointains, jamais >~15 mobs réels par groupe ; conçu pour 50-100 joueurs).

### Les 3 boucles de gameplay

- **🔥 Boucle courte (10-30 min, une expédition)** : sortir équipé du spawn → rejoindre
  une zone adaptée à son tier → piller les coffres LootRegion → tuer les mobs gênants
  (comptés par RegionStats) → rentrer déposer le butin avant de mourir.
- **⭐ Boucle moyenne (1-2 h, une incursion radioactive)** : préparer ses consommables
  (iode, filtres, soins) → entrer en zone radioactive (tier élevé) → survivre (gérer
  la dose, recharger la protection) → looter le matériel rare et les premiers artefacts
  → décontaminer au retour → crafter / inscrire le nouvel équipement.
- **☢ Boucle longue (plusieurs jours, end-game)** : rankup du survivant → capturer un
  territoire avec la faction → monter l'exosquelette + le filtre carbone (protection
  max) → accéder au tier 5 → récolter les artefacts dans les anomalies STALKER →
  débloquer les mutations (bonus permanents au prix d'expositions).

### Ton / direction artistique pour la création de contenu

- **Univers** : post-apo zombie, radiation, factions de survivants, ambiance STALKER /
  Tchernobyl. Vocabulaire : survivant, dose (mSv), signature/contamination, anomalie,
  artefact, hotspot, convoi, territoire, créneau, prime, capsule (cellule d'énergie).
- **Langue** : tout le contenu joueur est en **français**.
- **Format des messages** : MiniMessage avec une DA MaSurvie centralisée — balises
  `<primary>`, `<accent>`, `<prefix:main|error|success|warning|info>`. Le lang
  centralisé vit dans `plugins/SurvivorCore/lang_fr.yml` (~106 KB, sections
  namespacées par module). Les nombres exposés en placeholders doivent être des
  entiers (EpicAchievements rejette les décimaux).

---

## Sources lues pour ce fichier

- `wiki/concept_masurvie.md` (pitch, piliers, boucles, glossaire)
- `wiki/README.md` (hub plugins + changelog)
- `masurvie_mini_core/src/main/resources/plugin.yml` (identité, website, api-version)
- `masurvie_mini_core/pom.xml` (purpur-api 1.21.4, Java 21)
- divers `pom.xml` / `build.gradle` (versions Paper/Purpur, Java)

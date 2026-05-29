# 🧠 MaSurvie V2 — Cerveau de contexte

_Documentation de référence du serveur Minecraft **MaSurvie V2** (Skyblock + survie post-apocalyptique zombie, `masurvie.fr`)._

## À quoi sert ce dépôt

Ce dépôt **n'est pas du code** : c'est un **contexte créatif et conceptuel** destiné à être lu par un **assistant IA (Claude)**. Son but : permettre à l'assistant de **créer du contenu pour le serveur** — configs, succès, lore, dialogues, annonces Discord, menus, items — **dans le bon style et le bon univers**, sans qu'on doive tout réexpliquer à chaque fois.

Quand tu génères du contenu pour MaSurvie : **lis d'abord [`01-vue-ensemble.md`](01-vue-ensemble.md) et [`05-conventions-style.md`](05-conventions-style.md)**, puis le fichier thématique pertinent.

## 📑 Sommaire

| # | Fichier | Contenu |
|---|---------|---------|
| 1 | [01-vue-ensemble.md](01-vue-ensemble.md) | **Identité & philosophie** : nature du serveur (Skyblock + zombie), domaine, version, hébergeur, pitch, piliers, boucles de gameplay. **À lire en premier.** |
| 2 | [02-stack-plugins.md](02-stack-plugins.md) | **Stack de plugins** : tous les plugins installés, leur rôle, leur intégration. Séparés en code maison / autonomes / tiers. |
| 3 | [03-architecture-survivorcore.md](03-architecture-survivorcore.md) | **Architecture SurvivorCore** : pattern modulaire, chaque module un par un, le bus EventHub, les placeholders exposés (`%eventhub_counter_*%`, etc.). |
| 4 | [04-economie.md](04-economie.md) | **Économie** : provider Vault, monnaie (Capsules), comment on gagne/dépense, paliers de prime. |
| 5 | [05-conventions-style.md](05-conventions-style.md) | **Conventions de style (CRUCIAL)** : palette hex exacte, raretés Commun→Divin + couleurs, ItemsAdder (namespaces, model_data), format des lores, conventions de placeholders. |
| 6 | [06-style-annonces-discord.md](06-style-annonces-discord.md) | **Annonces Discord** : format maison des embeds (couleurs, structure, emojis, footer), déclencheurs auto, exemples complets. |
| 7 | [07-lore-univers.md](07-lore-univers.md) | **Lore / univers** : cadre narratif zombie, ton d'écriture, narration (Narrator/Flavortext), zones tier 0→5, grades, events spéciaux. |
| 8 | [08-contexte-divers.md](08-contexte-divers.md) | **Contexte transverse** : emplacements de fichiers, conventions (FR, MiniMessage, DB partagée), cheat-sheet de commandes, pièges connus. |
| 9 | [09-permissions.md](09-permissions.md) | **Permissions** : ~103 nœuds par module (avec leur défaut op/true/false). Utile pour les grades LuckPerms et savoir quelles features sont gardées. |

## ⚠️ Règle du dépôt (public)

Ce dépôt est **public**. Il ne contient **aucun secret** : pas d'identifiants MySQL, pas de token Discord, pas de clé API, pas d'URL de webhook. Les fichiers de secrets (`bdd.yml`, `bot/.env`, `survivor-panel/.env`, `webhooks.yml`) sont seulement **mentionnés par leur existence et leur rôle**, jamais par leur contenu. Le code source réel et premium reste dans un dépôt privé séparé.

> Quand tu ajoutes du contenu ici : **ne colle jamais de secret**, et **n'invente pas** de valeurs (IP, IDs Discord, identifiants). Si une donnée d'identité manque, laisse une mention « à confirmer par l'admin ».

---

_Source : dérivé du code et de la documentation interne du projet, le 2026-05-29._

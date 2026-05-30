# 04 — Économie du serveur MaSurvie V2

> Cerveau de contexte public. Ce fichier décrit **la monnaie**, **le provider Vault**,
> **comment on gagne/dépense**, et **les placeholders d'économie**. Toute IA qui crée
> du contenu touchant à l'argent (récompenses, primes, prix, lore de butin) doit
> respecter le nom et le ton de la monnaie décrits ici.

---

## 1 · Le provider d'économie

| Champ | Valeur |
|---|---|
| **Plugin d'économie** | **ExcellentEconomy** (auteur **nightexpress / nightcore**, même écurie qu'ExcellentCrates). API : `su.nightexpress.excellenteconomy.api.ExcellentEconomyAPI`. |
| **Pont** | **Vault** — ExcellentEconomy s'enregistre comme **provider Vault**. Tous les plugins maison lisent/écrivent l'argent **via Vault** (jamais l'API EE en direct côté code maison). |
| **Commande admin EE** | `eco give %player% <montant>` (ex. dans les récompenses AFK : `commands: ["eco give %player% 5"]`). C'est la commande EE standard. |
| **Lecture côté SurvivorCore** | Le module **PrimeTracker** lit le **solde Vault en read-only** pour calculer le palier de prime (cf. fichier 05 / wiki SurvivorCore). |

Source décisive : `masurvie_mini_core/src/main/resources/trade/currencies.yml` (commentaire
explicite « Vault → ExcellentEconomy via nightcore : 'capsules' (devise primaire MaSurvie) »)
et `modules/trade/absorbed/hooks/HookManager.java`.

> ⚠️ Les autres providers (CoinsEngine, EcoBits, MobCoins, UltraEconomy, PlayerPoints,
> TokenManager…) sont **présents dans le code AxTrade absorbé mais explicitement
> retirés / désactivés** car non installés côté serveur. **Ne jamais supposer une
> autre économie que Vault/ExcellentEconomy.**

---

## 2 · La monnaie : les CAPSULES

| Champ | Valeur |
|---|---|
| **Nom de la devise** | **Capsule(s)** — c'est la **monnaie unique** du serveur (devise primaire). |
| **Nom interne (currency name)** | `capsules` (déclaré dans `currencies.yml` sous le hook `Vault`). |
| **Emoji / symbole récurrent** | **💊** (pilule) — utilisé dans les affichages (`💊 Gains: +{...}`). Pas de symbole monétaire type `$`/`€`. |
| **Couleur d'affichage usuelle** | Souvent `<gold>` (jaune doré) pour les montants : `Capsules gagnees : <gold>%amount%`. En contexte gain positif on voit aussi `<green>`/`&a`. |
| **Lore univers** | Dans le lore du serveur (narrator.yml), « les CAPSULES, plus rares… » = la monnaie d'échange du monde post-apo (vivres, munitions, services). À traiter comme une **ressource de survie convoitée**, pas un argent abstrait. |

> ⚠️ Ne pas confondre **capsule (monnaie)** et **capsule/cellule d'énergie** (l'item
> qui alimente l'exosquelette, `cellule_energie`). Le **glossaire joueur** mélange un
> peu les deux termes, mais en économie « capsules » = **l'argent**.

### Exemples d'affichage réels (à copier-coller pour rester cohérent)

```yaml
# Convoi — part de butin versée
paid: "&a+{amount} &7Capsules (part de convoi, {members} membres)"

# Récap stats
stats_capsules_earned: "<gray>  Capsules gagnees : <gold>%amount%"

# Récompense AFK (commande EE)
- chance: 60
  display: "<gray>5 capsules"
  commands: ["eco give %player% 5"]
```

---

## 3 · La « Prime » — système de paliers (PrimeTracker)

Le module **PrimeTracker** de SurvivorCore ne crée **aucune monnaie** : il **lit le
solde Vault (capsules)** et en déduit un **palier (tier)** qui donne un bonus % et un
label coloré, exposés en placeholders. C'est un classement/statut, pas une 2ᵉ devise.

| ID | Solde (capsules) | Bonus | Label | Couleur hex |
|---|---|---|---|---|
| 0 | 0 – 499 | +0 % | **Réfugié** | `#AAAAAA` |
| 1 | 500 – 1999 | +10 % | **Survivant** | `#55FF55` |
| 2 | 2000 – 4999 | +20 % | **Chasseur** | `#55AAFF` |
| 3 | 5000 – 9999 | +35 % | **Prédateur** | `#FF55FF` |
| 4 | 10000 – ∞ | +50 % | **Légende** | `#FFAA00` |

Source : `deploy/configs/SurvivorCore/config.yml > prime-tracker.tiers`. Ces bornes et
labels sont **éditables** ; le `bonus` est juste un nombre exposé via PAPI, libre aux
autres systèmes (loot, drops) de l'appliquer.

> Le mot **« prime »** a deux sens dans l'univers : (1) le **palier/statut** ci-dessus,
> (2) une **récompense versée** (argent ou items) pour un objectif accompli (cf.
> glossaire). Les **créneaux** « Soirée Prime » boostent les gains/drops.

---

## 4 · Comment on gagne des capsules

D'après le concept serveur (FAQ « Comment je gagne de l'argent ? ») :

- **Piller des coffres LootRegion** puis **revendre le butin** (le loot des zones de
  tier élevé rapporte beaucoup plus).
- **Accomplir des objectifs** → **primes** versées par SurvivorCore.
- **Tenir un territoire** → **taxe perçue sur les pilleurs** présents dans la zone
  (TerritoryManager / module Territory).
- **Convois** : part de butin partagée entre membres du convoi (`+{amount} Capsules`).
- **Récompenses AFK** : petites sommes en zone AFK (`eco give`).
- **RegionStats rewards** : récompenses money/items configurables sur seuils de stats.

## 5 · Comment on dépense des capsules

- **`/trade`** (module Trade, ex-AxTrade absorbé) : échange joueur-à-joueur. Devises
  tradables = **capsules (Vault)** + **XP vanilla** uniquement. Tax configurable
  (défaut 0 %).
- **Boutiques / revente de butin** (mentionnées dans le concept ; le plugin de shop
  exact n'est pas dans le dépôt — voir « À confirmer » plus bas).
- **Rankup du survivant** (boucle longue ; mécanique de progression de grade).
- **Réparation à la Forge** (MaSurvie_Forgeron) : coût mixte **Vault + matériaux**.

---

## 6 · Placeholders d'économie

Pas d'expansion PAPI maison nommée « economy ». L'argent est exposé via :

| Placeholder | Source | Rendu |
|---|---|---|
| `%survivorcore_balance%` | PrimeTracker (snapshot Vault) | **entier** depuis 2026-05-29 (`Math.floor`), ex. `1500` |
| `%survivorcore_tier%` | PrimeTracker | ID numérique du tier (0–4) |
| `%survivorcore_tier_label%` | PrimeTracker | Libellé coloré (ex. `Survivant`) |
| `%survivorcore_bonus_percent%` | PrimeTracker | Bonus % du tier |
| `%survivorcore_bonus_multiplier%` | PrimeTracker | Multiplicateur (ex. `1.50`) |
| `%survivorcore_tier_next_missing%` | PrimeTracker | Capsules manquantes avant le palier suivant |
| `%survivorcore_rank%` / `%survivorcore_top_<pos>_amount%` | PrimeTracker | Classement primes |
| `%vault_eco_balance_fixed%` | **Vault PAPI** (pas maison) | Solde Vault entier. **À utiliser dans les YAML EpicAchievements** au lieu de `%vault_eco_balance%` (qui renvoie un décimal et fait spammer EA). |

Côté Trade, `%axtrade_*%` (Placeholders du module absorbé) peuvent exister mais ne
sont pas documentés comme placeholders maison de référence.

---

## Valeurs NON trouvées dans le dépôt (à confirmer par l'admin)

> ⚠️ À CONFIRMER : **le plugin de boutique / revente** (zAuctionHouse, EconomyShopGUI,
> Shop GUI Plus… ?). Le concept parle de « revente du butin » et de « boutique », mais
> **aucun config de shop** n'est présent dans le dépôt. AxTrade ≠ shop (c'est du
> trade joueur-à-joueur).
>
> ⚠️ À CONFIRMER : **le taux de revente / les prix** du butin (aucune table de prix
> trouvée dans le dépôt).
>
> ⚠️ À CONFIRMER : **le pluriel/affichage officiel d'ExcellentEconomy** (singulier
> « capsule » vs « capsules », format des grands nombres / séparateurs de milliers) —
> ces réglages vivent dans `plugins/ExcellentEconomy/` côté serveur, **hors dépôt**.

---

## Sources lues pour ce fichier

- `masurvie_mini_core/src/main/resources/trade/currencies.yml` (devise primaire = capsules / Vault → ExcellentEconomy)
- `masurvie_mini_core/.../modules/trade/absorbed/hooks/HookManager.java` (hooks retirés, Vault+XP seuls)
- `deploy/configs/SurvivorCore/config.yml` (paliers de prime, labels, couleurs hex)
- `wiki/concept_masurvie.md` (FAQ gains, taxe territoire, glossaire « prime »/« capsule »)
- `masurvie_mini_core/src/main/resources/afk/config.yml` (récompenses `eco give`, affichage capsules)
- `deploy/configs/SurvivorCore/lang_fr.yml` + `convoy/config.yml` (affichages 💊 capsules)
- `wiki/placeholders.md` + `wiki/survivorcore/README.md` (placeholders PrimeTracker, note `%vault_eco_balance_fixed%`)

---

## Prime au tueur - kill PvP (MAJ 2026-05-30)

Sur MaSurvie, la **prime** d'un joueur = **son solde Vault** (sa tete est mise a
prix a hauteur de ce qu'il possede). Depuis le 2026-05-30 :

- **Tuer un joueur en PvP transfere la TOTALITE de sa prime au tueur.** Le solde
  Vault de la victime est retire puis depose chez le tueur (transfert, pas
  creation de monnaie - neutre pour l'inflation).
- S'applique a **tout** kill PvP, **sauf** dans les **zones blacklistees**
  (regions WorldGuard sures : spawn, hub...), configurables.
- Game design : **plus tu es riche, plus tu es une cible.** Garder son argent sur
  soi devient risque -> incite a depenser/deposer vite et a tendre des embuscades
  sur les gros soldes. Coherent avec le leaderboard du top prime (PrimeTracker).

Implementation : module PrimeTracker, section config `prime-tracker.payout`
(`enabled`, `percentage`, `min-amount`, `notify`, `blacklist-regions`).

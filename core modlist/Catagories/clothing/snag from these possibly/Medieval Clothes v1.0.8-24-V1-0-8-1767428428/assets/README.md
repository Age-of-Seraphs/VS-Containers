# Assets — Structure et conventions

Ce document décrit la structure, les conventions et les bonnes pratiques pour le dossier `assets/` utilisé par le mod. Il vise à fournir une vue technique et pratique pour les auteurs de contenu (items, blocs, recettes, textures, patches, etc.).

---

## Objectif
- Expliquer l'organisation des fichiers sous `assets/`.
- Donner des conventions de nommage et des recommandations de validation.
- Fournir des exemples JSON (en anglais) utilisables comme référence.

---

## Vue d'ensemble

- `assets/` : racine des ressources du mod.
- `assets/game/` : (optionnel) copies ou références d'actifs du jeu de base.
- `assets/medieval/` : contenu principal du mod (items, blocs, recettes, shapes, textures, patches, langues).

Principaux sous-dossiers présents dans ce dépôt :

- `itemtypes/` — définitions déclaratives des objets (wearables, tools, resources).
- `blocktypes/` — définitions des blocs et leurs comportements.
- `recipes/` — recettes (`grid`, `smithing`, `barrel`, ...).
- `shapes/` — modèles 3D (entity, item, block).
- `textures/` — images (entity, item, block, hud, icons).
- `patches/` — JSON-Patch pour modifier les fichiers du jeu de base.
- `lang/` — fichiers de traduction (ex : `en.json`).

---

## Conventions générales

- Utiliser des noms de fichiers courts et explicites (`arm.json`, `canteen.json`).
- Référencer les assets avec un namespace : `medieval:...` ou `game:...`.
- Les fichiers originaux du moteur VintageStory peuvent utiliser une syntaxe proche de JSON5 (clefs non-quotées, commentaires, trailing commas). Pour la lecture automatique, privilégier un parseur JSON5 (library `json5`) ou un préprocesseur fiable.

---

## Dossier `itemtypes/`

But : définir les items et variantes (par ex. vêtements, outils, matériaux).

Pattern recommandé : `itemtypes/{category}/{subcategory}/file.json`.

Champs fréquents :

- `class` : ex. `"ItemWearable"` — comportement principal.
- `variantgroups` : génère plusieurs variantes à partir d'un seul fichier.
- `shapeByType` / `texturesByType` : mappages dynamiques pour variantes.

Bonnes pratiques :

- Grouper par catégorie (`wearable`, `tool`, `resource`).
- Référencer uniquement des `shapes` et `textures` existants.

Exemple (JSON, en anglais) :
```json
{
  "class": "ItemWearable",
  "code": "clothes-arm",
  "variantgroups": { "type": ["esclava", "bracers"] },
  "shapeByType": { "*-esclava": { "base": "medieval:item/wearable/arm/esclava" } },
  "texturesByType": { "*-esclava": { "base": "medieval:item/arm/esclava.png" } }
}
```

---

## Dossier `blocktypes/`

But : définir les blocs, leurs comportements et propriétés (attaches, containers, behaviors).

Champs courants : `behaviors`, `liquidContainerProps`, `attachableToEntity`.

Exemple (JSON, en anglais) :
```json
{
  "code": "medieval-canteen",
  "behaviors": ["GroundStorable", { "RightClickPickup": {} }],
  "attachableToEntity": {
    "categoryCode": "smallbag",
    "attachedShapeBySlotCode": { "smallbagl": { "base": "item/wearable/canteen-left" } }
  }
}
```

---

## Dossier `recipes/`

Types principaux : `grid`, `smithing`, `barrel`.

Grid recipe — champs typiques : `ingredientPattern`, `ingredients`, `width`, `height`, `output`.

Exemple (grid recipe, JSON, en anglais) :
```json
{
  "ingredientPattern": ["GGG"],
  "ingredients": { "G": { "item": "game:goldnugget" } },
  "width": 3,
  "height": 1,
  "output": { "item": "medieval:clothes-arm-esclava" }
}
```

---

## Dossier `shapes/`

Contient les modèles 3D pour entités, items et blocs.

Important : ces fichiers sont sensibles — évitez les modifications automatiques. Si vous devez les éditer, faites des sauvegardes.

Référencer les shapes via `namespace:path`, exemple : `medieval:entity/humanoid/seraph/clothing/arm/esclava1`.

---

## Dossier `textures/`

Organisation : `textures/entity/`, `textures/item/`, `textures/block/`, `textures/hud/`, `textures/icons/`.

Conseils :

- Nommez les images clairement, conservez des tailles cohérentes (32x32, 64x64, etc.).
- Ne modifiez pas les PNG par script sans sauvegarde.

---

## Dossier `patches/`

But : stocker des documents JSON-Patch destinés à modifier des assets du jeu de base.

Format d'opération (exemple en anglais) :
```json
[
  {
    "op": "add",
    "path": "/drops/0",
    "value": { "item": "medieval:rabbit", "prob": 0.5 },
    "file": "game:entities/animal/hare-adult.json"
  }
]
```

Bonnes pratiques :

- Toujours valider les chemins cibles (`file`) avec une copie de `VintageStory-Original`.
- Tester en jeu avant publication.

---

## Dossier `lang/`

Contient les traductions au format JSON clé-valeur (ex : `en.json`).

Exemple (JSON, en anglais) :
```json
{
  "item-medieval:clothes-arm-esclava": "Esclava Bracer",
  "block-medieval:medieval-canteen": "Canteen"
}
```

---

## Recommandations transversales

- Préférer `json5` pour parser les fichiers originaux du jeu (ils peuvent contenir des commentaires, des clefs non-quotées et des trailing commas).
- Utiliser des chemins namespace-qualifiés (`medieval:...`, `game:...`).
- Garder une convention de variants cohérente pour permettre l'utilisation de `shapeByType` / `texturesByType` avec des jokers (`*-esclava`).

---

## Erreurs fréquentes et solutions

- `Expecting property name enclosed in double quotes` : fichier JSON5-like — utilisez `json5` ou un préprocesseur pour enlever commentaires/quotage.
- `Missing texture/shape` : mauvaise référence dans `shapeByType` ou `texturesByType` — vérifier les chemins et les noms de fichiers.
- `Patch ignored` : `file` ou `path` invalide dans le patch — vérifier la structure exacte dans `VintageStory-Original`.

---

## Checklist avant commit

- Tous les JSON parsés (utiliser `json5` si nécessaire).
- Tous les chemins `shape` / `texture` existent.
- Les patches ciblent des fichiers présents dans `VintageStory-Original`.
- Les clés de langue sont renseignées.
- Sauvegardes pour les shapes/textures modifiées.

---

## Ressources utiles

- `VintageStory-Original/` : source de vérité pour la structure des fichiers.
- `json5` : parser recommandé pour JSON5-like inputs.
- RFC 6902 — JSON Patch (concepts pour `op`, `path`, `value`).

---

Si vous souhaitez, je peux :

- Générer ce README directement dans `assets/` (fait).
- Ajouter des exemples ciblés en scannant le contenu actuel de `assets/` et en produisant une carte des fichiers.
- Mettre à jour `requirements.txt` pour ajouter `json5`.

---

Fini.

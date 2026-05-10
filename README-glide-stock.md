# App Glide « Stock MGT » — interface demandée

Ce dossier contient une base **prête à importer dans Glide** pour reproduire l'interface montrée dans les captures:

- écran unique de navigation: **Saisie Mouvement**;
- affichage principal en **Table**;
- bouton bleu **+ Add** pour ajouter un mouvement;
- formulaire « Ajouter un élément » avec les champs du mouvement;
- écran détail « Mouvement du ... » avec boutons **Générer Bon** et **Éditer**;
- liste des produits liés au mouvement avec bouton **+ Add**;
- formulaire produit avec **Produit**, **Quantité**, **Marque**, **Image du Mouvement**;
- écran bon imprimable: **Bon de Réception**, **Bon de Livraison** ou **Bon de Retour** selon le type de mouvement.

## Fichiers du pack

| Fichier | À quoi il sert |
| --- | --- |
| `mouvements.csv` | Table principale `Mouvements de Stock` pour la liste et le formulaire de mouvement. |
| `lignes_mouvement.csv` | Table enfant `Lignes Produits` pour saisir plusieurs produits dans une même opération. |
| `produits.csv` | Référentiel produit optionnel pour alimenter les listes déroulantes. |
| `glide-app-spec.json` | Plan technique de l'interface Glide: écrans, composants, colonnes et actions. |
| `bon-template.html` | Aperçu HTML imprimable du bon, proche de l'écran montré dans la capture. |

## 1) Importer les données dans Glide

1. Dans Glide, cliquer sur **Create app**.
2. Choisir **Import File**.
3. Importer `mouvements.csv` et renommer la table: **Mouvements de Stock**.
4. Ajouter/importer `lignes_mouvement.csv` et renommer la table: **Lignes Produits**.
5. Ajouter/importer `produits.csv` et renommer la table: **Produits**.

## 2) Table principale: `Mouvements de Stock`

Colonnes à avoir dans Glide:

| Colonne | Type Glide conseillé | Utilisation dans l'interface |
| --- | --- | --- |
| `ID_MOUVEMENT` | Row ID / Text | Identifiant unique du mouvement. |
| `Date` | Date Time | Première colonne de la table et titre du détail. |
| `Type de Mouvement` | Choice | Liste déroulante: `Entrée`, `Sortie`, `Retour`. |
| `Nom de la Personne` | Text Entry | Personne responsable du mouvement. |
| `Client/Fournisseur` | Text Entry | Client pour sortie/retour, fournisseur pour entrée. |
| `Service` | Text Entry | Service concerné. |
| `Sous-Traitant` | Text Entry | Sous-traitant éventuel. |
| `Image du Mouvement` | Image Picker | Image ou justificatif. |
| `Type de Bon` | If-Then-Else / Text | Bon généré automatiquement selon le mouvement. |
| `Numero Bon` | Template / Text | Numéro du bon. |
| `pdf_url` | URL | Lien PDF si génération externe. |
| `Statut` | Choice | Brouillon, Validé, Imprimé. |

## 3) Table enfant: `Lignes Produits`

Cette table permet de saisir **plusieurs produits pour une seule opération**.

| Colonne | Type Glide conseillé | Utilisation |
| --- | --- | --- |
| `ID_LIGNE` | Row ID / Text | Identifiant unique de la ligne. |
| `ID_MOUVEMENT` | Text, valeur cachée | Reçoit automatiquement l'ID du mouvement en cours. |
| `Produit` | Text Entry ou Choice | Désignation du produit. |
| `Quantité` | Number Entry | Quantité entrée/sortie/retournée. |
| `Marque` | Text Entry | Marque du produit. |
| `Image du Mouvement` | Image Picker | Photo optionnelle de la ligne produit. |

## 4) Colonnes calculées à créer dans Glide

Dans la table **Mouvements de Stock**, créer ces colonnes calculées:

1. **Relation Produits du Mouvement**
   - Type: `Relation`
   - Colonne locale: `ID_MOUVEMENT`
   - Table cible: `Lignes Produits`
   - Colonne cible: `ID_MOUVEMENT`
   - Relation multiple: activée.

2. **Type de Bon automatique**
   - Type: `If-Then-Else`
   - Si `Type de Mouvement` = `Entrée` → `Bon de Réception`
   - Si `Type de Mouvement` = `Sortie` → `Bon de Livraison`
   - Si `Type de Mouvement` = `Retour` → `Bon de Retour`

3. **Nombre Produits**
   - Type: `Rollup`
   - Source: `Relation Produits du Mouvement`
   - Opération: `Count`.

4. **Total Quantités**
   - Type: `Rollup`
   - Source: `Relation Produits du Mouvement > Quantité`
   - Opération: `Sum`.

## 5) Écran principal comme la capture: `Saisie Mouvement`

Dans l'onglet **Layout**:

1. Créer une seule navigation: **Saisie Mouvement**.
2. Source: **Mouvements de Stock**.
3. Style: **Table**.
4. Colonnes visibles, dans cet ordre:
   - `Date`
   - `Type de Mouvement`
   - `Nom de la Personne`
   - `Client/Fournisseur`
   - `Service`
   - `Sous-Traitant`
   - `Image du Mouvement`
5. Dans **Actions**, activer:
   - `Allow users to add items`;
   - `Allow users to edit items`.

## 6) Formulaire `+ Add` pour ajouter un mouvement

Le formulaire de la table **Mouvements de Stock** doit contenir ces composants, dans cet ordre:

1. `Date Time` → colonne `Date`.
2. `Choice` → colonne `Type de Mouvement` avec les options:
   - `Entrée`
   - `Sortie`
   - `Retour`
3. `Text Entry` → colonne `Nom de la Personne`.
4. `Text Entry` → colonne `Client/Fournisseur`.
5. `Text Entry` → colonne `Service`.
6. `Text Entry` → colonne `Sous-Traitant`.
7. `Image Picker` → colonne `Image du Mouvement`.

Réglage recommandé:

- titre du formulaire: **Ajouter un élément**;
- bouton de validation: **Soumettre**;
- après soumission: notification **Mouvement enregistré**.

## 7) Écran détail du mouvement

Quand l'utilisateur clique sur une ligne de la table, créer l'écran détail suivant:

1. **Title**
   - Titre: `Mouvement du {Date}`
   - Sous-titre: `{Type de Mouvement}`

2. **Button Block**
   - Bouton 1: **Générer Bon**
     - action: ouvrir l'écran Bon ou ouvrir `pdf_url` si PDF généré.
   - Bouton 2: **Éditer**
     - action: `Edit this item`.

3. **Fields** avec le titre **Informations du Mouvement**
   - `Date`
   - `Type de Mouvement`
   - `Nom de la Personne`
   - `Client/Fournisseur`
   - `Service`
   - `Sous-Traitant`

4. **Inline List / Collection**
   - Source: `Relation Produits du Mouvement`
   - Style: List
   - Titre de ligne: `Produit`
   - Sous-titre: `Quantité`
   - Image: `Image du Mouvement`.

5. Bouton **+ Add** pour ajouter un produit au mouvement.

## 8) Formulaire produit `+ Add`

Le formulaire doit écrire dans **Lignes Produits**.

Champs visibles:

1. `Text Entry` ou `Choice` → `Produit`.
2. `Number Entry` → `Quantité`.
3. `Text Entry` → `Marque`.
4. `Image Picker` → `Image du Mouvement`.

Valeur cachée obligatoire:

- `ID_MOUVEMENT` = valeur de l'écran actuel `Mouvements de Stock > ID_MOUVEMENT`.

C'est ce réglage qui permet d'ajouter 2, 3, 10 produits dans une seule opération, sans recréer la date, le type, le client ou le service.

## 9) Écran Bon / Impression

Créer un écran ou un document avec:

- en-tête: **SARL MGT (Medical Gas Technologie)**;
- sous-titre: `Importation, Fabrication et Installation fluides médicaux`;
- logo MGT;
- grand titre dynamique:
  - `Bon de Réception` pour `Entrée`;
  - `Bon de Livraison` pour `Sortie`;
  - `Bon de Retour` pour `Retour`;
- champs:
  - `Date`;
  - `Fournisseur / Client`;
- tableau des produits liés:
  - `DESIGNATION` = `Produit`;
  - `QTE` = `Quantité`.

Le bouton **Imprimer** peut utiliser:

- `Open link` vers la colonne `pdf_url`, si tu génères un PDF avec Make/Zapier/PDFMonkey;
- ou un écran Glide formaté comme bon, puis impression depuis le navigateur.

## 10) Résultat attendu

Avec cette structure, l'app fonctionne comme dans les captures:

1. l'utilisateur arrive sur **Saisie Mouvement**;
2. il clique **+ Add**;
3. il remplit Date, Type de Mouvement, Nom, Client/Fournisseur, Service, Sous-Traitant et image;
4. il ouvre le mouvement créé;
5. il ajoute plusieurs lignes produit avec Produit, Quantité, Marque;
6. il clique **Générer Bon**;
7. l'app affiche ou ouvre le bon correspondant: réception, livraison ou retour.

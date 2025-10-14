
# Projet Qualité des Données : Compte Rendu

**Réalisé par :**  
- TAHIRI Mouad  
- NOURI Yassir  
- HARGAS Ali  
- BARDI Mohamed Ali  

---

## 🧩 Première étape

Cette première étape du projet vise à concevoir les **mappings logiques** permettant d’évaluer la cohérence des données issues des différentes sources du projet :

> *“Évaluation et amélioration de la qualité de données de consommation énergétique”*.

L’objectif est d’identifier comment les **attributs des sources (S1 à S4)** s’articulent pour produire les **tables cibles**.

---

## 🏠 Normalisation et décomposition d’adresses

Les adresses apparaissent dans plusieurs tables (`Consommation`, `Population`, `IRIS`), mais sous des formats différents.  
Pour permettre des **jointures fiables** entre ces sources, il faut d’abord les **décomposer** et **normaliser**.

---

### Étape 1 : Découpage d’adresse complète

**Sources :**  
- `Population.Adresse`  
- `Consommation.Adresse`

**Objectif :**  
Extraire les éléments suivants :

- Numéro de rue → `Numero_Rue`  
- Nom de la rue → `Nom_Rue`  
- Code postal → `Code_Postal`  
- Ville (optionnelle) → `Ville`

**Table d’exemples :**

| Champ         | Description                          | Exemple                        |
|----------------|--------------------------------------|--------------------------------|
| `Numero_Rue`  | Numéro + suffixe éventuel (BIS, TER) | 12BIS                          |
| `Nom_Rue`     | Nom normalisé de la voie             | AVENUE DU GENERAL LECLERC      |
| `Code_Postal` | Code postal à 5 chiffres             | 75013                          |

**Exemple de transformation :**

```
"12 bis av. du Général Leclerc 75013"
→ Numero_Rue = 12BIS
→ Nom_Rue = AVENUE DU GENERAL LECLERC
→ Code_Postal = 75013
```

---

### Étape 2 : Nettoyage lexical et standardisation

**But :** garantir que les adresses ont le même format dans toutes les tables.

**Règles appliquées :**

| Règle | Exemple |
|-------|----------|
| Supprimer les espaces superflus | `" AV . Victor Hugo "` → `"AV. VICTOR HUGO"` |
| Mettre en majuscules | `"rue de Paris"` → `"RUE DE PARIS"` |
| Supprimer les accents | `"É"` → `"E"`, `"Ç"` → `"C"` |
| Uniformiser les abréviations | `"AV."` → `"AVENUE"`, `"BD"` → `"BOULEVARD"` |
| Supprimer ponctuations et points | `"AV. VICTOR-HUGO"` → `"AVENUE VICTOR HUGO"` |
| Normaliser les numéros | `"12 bis"` → `"12BIS"` |

---

### Étape 3 : Création d’une clé d’adresse normalisée

**But :** obtenir une **clé unique** pour les jointures entre tables.

**Transformation :**

```
Adresse_Normalisee = concat(Numero_Rue, "_", Nom_Rue, "_", Code_Postal)
```

**Exemple :**

```
"12BIS_AVENUE DU GENERAL LECLERC_75013"
```

Cette clé est calculée dans :
- `Population`
- `Consommation`
- `IRIS` (si disponible)

Elle sert à effectuer des **jointures précises**.

---

## 🔗 Jointures entre tables

### Jointure `Consommation` ↔ `IRIS`

**Objectif :**  
Associer chaque adresse de consommation à une zone géographique IRIS.

**Conditions de jointure :**

```sql
normalize(Consommation.Nom_Rue) = IRIS.ID_Rue
AND Consommation.Code_Postal = IRIS.Code_Postal
AND IRIS.ID_Ville IN ('Paris', 'Evry')
```

**Cas particuliers :**

- Si `Nom_Rue` est absent → fallback sur `Code_Postal + Ville`  
- Si plusieurs IRIS possibles → choisir le plus fréquent  
- Si aucun match → `ID_IRIS = NULL`

---

### Jointure `Population` ↔ `Consommation`

**Objectif :**  
Relier les individus à leurs consommations par adresse.

**Condition de jointure :**

```sql
Population.Adresse_Normalisee = Consommation.Adresse_Normalisee
```

**Cas gérés :**

- Si plusieurs foyers sur une même adresse → consommation moyenne (`AVG`)  
- Si une adresse présente dans `Consommation` mais absente dans `Population` → ligne ignorée

---

### Jointure `Population` ↔ `CSP`

**Objectif :**  
Associer chaque individu à sa **catégorie socio-professionnelle**.

**Condition :**

```sql
Population.CSP = CSP.ID_CSP
```

---

## 📊 Calculs d’agrégation

| Type         | Transformation                  | Cible                  | Objectif                                      |
|---------------|----------------------------------|-------------------------|-----------------------------------------------|
| **SUM annuelle** | `SUM(NB_KW_Jour * 365)`       | `Consommation_IRIS_*`  | Total de consommation annuelle par IRIS       |
| **AVG annuelle** | `AVG(NB_KW_Jour * 365)`       | `Consommation_CSP`     | Moyenne annuelle de consommation par CSP      |

---



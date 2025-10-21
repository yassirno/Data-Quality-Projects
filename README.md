
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

_________________________________
Consommation IRIS Paris
T1_Paris — Agréger la consommation par rue et code postal
T1_Paris = γ Nom_Rue, Code_Postal; SUM(NB_KW_Jour) → Total_KW (Consommation)

T2_Paris — Jointure entre Consommation et IRIS
T2_Paris = T2_Paris ⋈ (T2_Paris.Nom_Rue = IRIS.ID_Rue) IRIS

T3_Paris — Filtrer sur la ville “Paris”
T3_Paris = σ (IRIS.ID_Ville = 'Paris')(T2_Paris)

Consommation_IRIS_Paris = γ ID_IRIS; (SUM(Total_KW) × 365) → Conso_moyenne_annuelle (T3_Paris)
_________________________________
Consommation IRIS Evry


T1_Evry — Agréger la consommation par rue et code postal
T1_Evry = γ Nom_Rue, Code_Postal; SUM(NB_KW_Jour) → Total_KW (Consommation)

T2_Evry — Jointure entre Consommation et IRIS
T2_Evry = T1_Evry ⋈ (T1_Evry.Nom_Rue = IRIS.ID_Rue) IRIS

T3_Evry — Filtrer sur la ville “Evry”
T3_Evry = σ (IRIS.ID_Ville = 'Evry')(T2_Paris)

Consommation_IRIS_Evry = γ ID_IRIS; (SUM(Total_KW) × 365) → Conso_moyenne_annuelle (T3_Evry)


_________________________________
Consommation CSP
Population(ID_Personne, Nom, Prénom, N , nom_Rue, code_postal, CSP)
on a fait la separation de l’adresse a l’étape 1


T1_conso_pop— Jointure entre Consommation et population
T1-conso_pop = Consomation ⋈    Population
				(Consomation.nom_rue= Population.nom_Rue)
T2_Conso_pop — group by
Conso_pop=  γ(csp, nom_Rue, SUM(NB_KW_Jour) → Total_KW ) (T1-conso_pop)

T3-Consommation_CSP  — jointure Conso_pop
Consommation_CSP = Conso_pop ⋈    CSP
			(conso_pop.CSP= CSP.id_csp)


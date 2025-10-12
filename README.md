# G8: Tahiri Mouad, Hargas Ali, Nouri Yassir, Bardi Mohamed Ali

## 🧩 Contexte du projet

Cette première étape du projet vise à **concevoir le mapping logique** permettant d’évaluer la **qualité et la cohérence des données** issues des différentes sources du projet **« Consommation énergétique »**.  
L’objectif est d’identifier comment les attributs des sources (**S1 à S4**) s’articulent pour produire les **tables cibles**, et d’exprimer ces correspondances à l’aide des **opérateurs de l’algèbre relationnelle**.

---

## 🎯 Objectifs de l’étape

- Identifier les **correspondances** entre les champs des **tables sources** et les **tables cibles**.  
- Définir les **opérations de jointure**, **filtrage** et **agrégation** nécessaires à la construction des données cibles.  
- Préparer la **future automatisation** de ces opérations dans un **workflow Airflow**.  
- Déterminer les **vérifications de qualité** à effectuer sur les sources de données.  
### Mapping 1 – Consommation_IRIS_Paris
But : Calculer la *consommation moyenne annuelle* par zone IRIS pour la ville de Paris. On
utilise la table des consommations de Paris et on la relie à la table IRIS pour identifier la zone
géographique de chaque rue.
Formules algébriques :
T1_paris = Consommation_Paris ⋈ (Consommation_Paris.N = IRIS.ID_Rue) IRIS
T2_paris = σ (IRIS.ID_Ville = 'Paris')(T1_paris)
Consommation_IRIS_Paris = γ ID_IRIS, (AVG(NB_KW_Jour) × 365) → Conso_moyenne_annuelle
(T2_paris)
Vérifications : correspondance entre rues et IRIS, valeurs manquantes ou négatives, cohérence
des unités de consommation.
### Mapping 2 – Consommation_IRIS_Evry
But : Calculer la consommation moyenne annuelle par zone IRIS pour la ville d’Évry. La logique est
identique à celle de Paris, en filtrant sur la ville 'Evry'.
Formules algébriques :
T1_evry = Consommation_Evry ⋈ (Consommation_Evry.N = IRIS.ID_Rue) IRIS
T2_evry = σ (IRIS.ID_Ville = 'Evry')(T1_evry)
Consommation_IRIS_Evry = γ ID_IRIS, (AVG(NB_KW_Jour) × 365) → Conso_moyenne_annuelle
(T2_evry)
Vérifications : taux de jointure correct, absence de doublons d’IRIS, contrôle des valeurs
anormales.
### Mapping 3 – Consommation_CSP
But : Évaluer la consommation annuelle moyenne par *catégorie socio-professionnelle (CSP)* et
l’associer au *salaire moyen* correspondant. Les données de population et de consommation
sont combinées pour Paris et Évry, puis reliées à la table CSP.
Formules algébriques :
T5 = Population_Paris ⋈ (Population_Paris.Adresse = Consommation_Paris.ID_Adr)
Consommation_Paris
T6 = Population_Evry ⋈ (Population_Evry.Adresse = Consommation_Evry.ID_Adr)
Consommation_Evry
T7 = T5 ∪ T6
T8 = γ CSP, (AVG(NB_KW_Jour) × 365) → Conso_moyenne_annuelle (T7)
Consommation_CSP = T8 ■ (T8.CSP = CSP.ID_CSP) CSP
Vérifications : existence des correspondances Adresse–ID_Adr, CSP manquants, valeurs nulles ou
négatives dans NB_KW_Jour, unicité des identifiants CSP.
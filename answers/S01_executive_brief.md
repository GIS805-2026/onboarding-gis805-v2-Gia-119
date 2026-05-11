# Board Brief — S01
Préparer par le Head of Data

## Question du CEO
Le CEO a besoin de connaitre quel revenu par catégorie de produits et par région des magasins les plus performantes en termes de revenus afin d’aider à prendre des décisions stratégiques.

## Réponse exécutive
L’analyse des ventes montre que le revenu varie fortement selon la catégorie de produits et la région.
Les catégories Toys & Games, Automotive, Grocery, Books & Media et Beauty & Health génèrent les revenus les plus élevés, principalement au Québec et en Ontario, qui sont les principaux marchés.
À l’inverse, des catégories comme Clothing, Electronics et Home & Garden génèrent des revenus plus faibles et contribuent marginalement au chiffre d’affaires total, notamment dans les régions Outaouais, Estrie, Alberta et BC.

Le chiffre d’affaires est donc concentré à la fois par catégorie et par région, avec quelques combinaisons dominant largement les autres.

## Décisions de modélisation
Problème identifié →  Décision Kimball
─────────────────────────────────────
Données transactionnelles brutes 
(orders + order_lines éparpillés)
                     →  Grain : une ligne de commande
                         (order_number, line_id)

Mesures non calculées 
(pas de revenu unit / total)
                     →  Créer line_total = quantity × unit_price

Catégorie dispersée 
dans dim_product
                     →  Dimension conforme product 
                         avec colonne category

Région du magasin 
dans dim_store
                     →  Dimension conforme store 
                         avec colonne region


## Preuve
1. Requête de preuve :
    SELECT
    p.category      AS product_category,
    s.region        AS store_region,
    SUM(f.line_total) AS revenue
FROM raw_fact_sales f
JOIN raw_dim_product p ON f.product_id = p.product_id
JOIN raw_dim_store   s ON f.store_id   = s.store_id
GROUP BY
    p.category,
    s.region
ORDER BY
    revenue DESC;

Elle joint le fait de vente (raw_fact_sales) avec les dimensions métier pertinentes.
Elle agrège la mesure additive line_total.
Elle délivre exactement le KPI demandé : revenu par catégorie de produit et par région de magasin.
Cela montre que le grain choisi (ligne de commande) est suffisant et stable.

Après exécution :

La première ligne du résultat (catégorie + région + revenu).
La tendance principale identifiée (voir réponse dans ## Réponse exécutive).

## Validation
J’ai vérifié que les tables utilisées existent dans DuckDB : raw_fact_sales, raw_dim_product, raw_dim_store.
J’ai vérifié que les colonnes métier existent et sont utilisables : raw_fact_sales.line_total, raw_dim_product.category, raw_dim_store.region.
J’ai exécuté la requête de preuve sur raw_fact_sales, raw_dim_product et raw_dim_store, et j’ai confirmé que line_total est bien la mesure de revenu. J’ai aussi vérifié l’existence des clés et l’absence de valeurs nulles critiques avant d’agréger.

## Risques / limites
Limites des données brutes : les tables raw_* ne sont pas optimisées pour les analyses répétées ; les jointures sont lentes  et risquent des erreurs si les clés changent.
Qualité des données : potentiels doublons, valeurs manquantes dans line_total ou clés étrangères orphelines.
Portée limitée : cette analyse ne couvre que les ventes, pas les retours, inventaires ou budgets — le CEO pourrait vouloir plus.
Grain assumé : si une commande a plusieurs lignes, le revenu par région pourrait être surévalué si on ne dédoublonne pas correctement.

## Prochaine recommandation
Construire le schéma en étoile avec fact_sales au grain défini, et les dimensions conformes dim_product, dim_store, etc.
Implémenter les clés de substitution (product_key, store_key) pour éviter les problèmes de clés naturelles.
Ajouter des dimensions temporelles (dim_date) pour permettre des analyses par trimestre.

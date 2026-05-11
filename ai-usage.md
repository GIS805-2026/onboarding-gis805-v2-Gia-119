# Trace d'usage IA — GIS805

> Chaque interaction significative avec un outil IA doit être documentée ici.
> Ce fichier est **obligatoire** et évalué à chaque remise.

## Format par entrée

```
### YYYY-MM-DD — Séance SXX
- **Modèle :** (ChatGPT-4o, Claude, Copilot, etc.)
- **Prompt :** (copier-coller exact)
- **Résultat :** (résumé de ce que l'IA a produit)
- **Validation :** (comment vous avez vérifié/modifié le résultat)
- **Justification :** (pourquoi cette interaction était nécessaire)
```

---

### 2026-05-10 — Séance S02 
- **Modèle :** GitHub Copilot Chat
- **Prompt :** « 1.Quelles tables existent dans ma base de données DuckDB ? 2.Montre-moi les colonnes de la table orders ? 3.Combien de clients, de commandes, et de produits j'ai ? 4.Ecris une requête pour voir le revenue par catégorie par région ? »
- **Résultat : 1.duckdb db/nexamart.duckdb -c "SHOW TABLES;" = 24 tables  2.duckdb db/nexamart.duckdb -c "DESCRIBE raw_orders;"  3.duckdb db/nexamart.duckdb -c "SELECT (SELECT COUNT(DISTINCT customer_id) FROM raw_dim_customer) AS nombre_clients_uniques, (SELECT COUNT(*) FROM raw_fact_sales) AS nombre_commandes, (SELECT COUNT(DISTINCT product_id) FROM raw_dim_product) AS nombre_produits_uniques;" Comptes dans votre dataset : clients : 193, produits : 65, commandes : 2590  4.duckdb db/nexamart.duckdb -c "SELECT p.category, s.region, SUM(f.line_total) AS revenue FROM fact_sales f JOIN dim_product p ON f.product_id = p.product_id JOIN dim_store s ON f.store_id = s.store_id GROUP BY p.category, s.region ORDER BY revenue DESC;"   
- **Validation :** J’ai vérifié que les tables utilisées existent dans DuckDB : raw_fact_sales, raw_dim_product, raw_dim_store. J’ai vérifié que les colonnes métier existent et sont utilisables : raw_fact_sales.line_total, raw_dim_product.category, raw_dim_store.region, J’ai vérifié la qualité minimale des données : SELECT COUNT(*) FROM raw_fact_sales; SELECT COUNT(*) FROM raw_fact_sales WHERE line_total IS NULL; SELECT COUNT(*) FROM raw_fact_sales WHERE product_id NOT IN (SELECT product_id FROM raw_dim_product); SELECT COUNT(*) FROM raw_fact_sales WHERE store_id NOT IN (SELECT store_id FROM raw_dim_store);
- **Justification :** Je voulais comprendre les différentes tables et données à l'intérieurs, avoir une vue pour une meilleur analyse des données.

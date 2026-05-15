# Rétroaction automatisée -- S01 (Diagnostic fondamental -- NexaMart kickoff)

_Générée le 2026-05-14T22:18:17+00:00 -- Run `20260514T221333Z-7d34bf6a`_

Ce document est produit par un pipeline reproductible (vérification SQL déterministe + analyse LLM du brief et de la déclaration IA). Une revue humaine précède toujours sa publication. **À ce stade expérimental, aucune note ni étiquette de niveau n'est diffusée : l'objectif est purement formatif.**

---

## 1. Vérification automatique de la requête SQL

La requête extraite de votre brief n'a pas pu être validée automatiquement. Quelques pistes constructives ci-dessous pour vous aider à la rendre exécutable et alignee avec la question posée.

_Observation technique : colonnes manquantes (oracle): quarter_

<details><summary>Requête analysée — cliquez pour déplier</summary>

```sql
WITH revenue_by_month AS ( SELECT p.category, s.region, d.year, d.month, SUM(f.line_total) AS revenue FROM raw_fact_sales f JOIN raw_orders o ON f.order_number = o.order_number JOIN raw_dim_product p ON f.product_id = p.product_id JOIN raw_dim_store s ON f.store_id = s.store_id JOIN raw_dim_date d ON o.order_date = d.date_key GROUP BY p.category, s.region, d.year, d.month ), revenue_with_lag AS ( SELECT category, region, year, month, revenue, LAG(revenue) OVER ( PARTITION BY category, region ORDER BY year, month) AS previous_revenue, revenue - LAG(revenue) OVER (PARTITION BY category, region ORDER BY year, month) AS revenue_change FROM revenue_by_month) SELECT * FROM revenue_with_lag WHERE previous_revenue IS NOT NULL ORDER BY revenue_change ASC;
```

</details>

- Colonnes retournées : `category, region, year, month, revenue, previous_revenue, revenue_change`
- Correspondance avec les colonnes attendues :
  - `category` → `category`
  - `region` → `region`
  - `quarter` → `(à ajouter ou renommer)`
  - `revenue` → `revenue`

**Pistes :**
> Requête extraite par LLM (aucun bloc fencé détecté). Encadrez votre requête finale par ```sql ... ``` pour éliminer toute ambiguïté.
> Votre `db/nexamart.duckdb` est absente ou vide ; la requête a été exécutée contre une **base de référence cohorte** (seed instructeur). Les chiffres retournés ne correspondent donc pas à vos propres données : reconstruisez votre base avec `python src/run_pipeline.py` (ou `.\run.ps1 load`) pour valider vos calculs sur votre seed personnel.
> Synonymes acceptés par colonne:
  category: ['category', 'categorie', 'p.category', 'sous_categorie']
  region: ['region', 's.region']
  quarter: ['quarter', 'trimestre', 'd.quarter', 'q']
  revenue: ['net_revenue', 'revenue', 'revenu', 'total_revenue', 'ca', 'sales', 'line_total', 'gross_revenue']

## 2. Rétroaction pédagogique sur le brief

> Bon diagnostic opérationnel avec grain clair et une requête de preuve fonctionnelle; le brief identifie correctement catégories et régions prioritaires. Pour monter en excellence, documentez la traçabilité (commits, note IA) et renforcez la justification décisionnelle (SCD, KPI chiffrés, checks reproductibles).

### Observations par dimension

**Model quality**
- Observation : Le brief déclare explicitement le grain (ligne de commande), propose la création de line_total = quantity × unit_price et un schéma en étoile Kimball avec dim_product et dim_store.
- Piste d'amélioration : Préciser le traitement SCD (type 2) pour dim_product/dim_store et justifier le choix du pattern par rapport aux changements historiques.

**Validation quality**
- Observation : Le document inclut une requête DuckDB qui calcule revenue_by_month et utilise LAG() pour mesurer la variation de revenu par catégorie et région.
- Piste d'amélioration : Ajouter des contrôles explicites pour NULLs, doublons du grain et tests de cohérence (somme des line_total vs total attendu) et montrer les résultats des checks.

**Executive justification**
- Observation : La section exécutive identifie les catégories et régions dominantes (ex. Toys & Games, Québec et Ontario) et conclut que le chiffre d'affaires est concentré, utile pour la prise de décision.
- Piste d'amélioration : Formuler une recommandation décisionnelle plus précise (ex. prioriser X régions/catégories pour investissement ou réduction de l'assortiment) et ajouter un ou deux KPI chiffrés à l'appui.

**Process trace**
- Observation : Le brief indique que la requête a été exécutée et que certaines vérifications ont été faites, mais il n'y a aucune mention d'un historique de commits ni d'une note IA détaillée.
- Piste d'amélioration : Fournir un log de commits git (≥3 commits) avec messages explicites et ajouter une note IA précisant outil, prompt utilisé et validation humaine.

**Reproducibility**
- Observation : La preuve montre la commande duckdb db/nexamart.duckdb -c "..." mais le chemin de la base est codé en dur et le README/steps de reproduction ne sont pas fournis.
- Piste d'amélioration : Ajouter un README clair avec étapes 'clone → installer dépendances → exécuter make check' et éviter les chemins codés en dur (ou documenter les ajustements nécessaires).

## 3. Déclaration d'utilisation de l'IA

> La déclaration documente l'outil utilisé, l'étape (Séance S02) et donne une validation humaine détaillée via requêtes SQL. En revanche, elle ne mentionne pas de version/modèle précis pour Copilot ni les limites ou erreurs observées, ce qui empêche une conformité complète.

**Sujets bien couverts dans votre déclaration :**

- outils utilisés (nom + version/modèle)
- à quelle étape l'IA a été utilisée
- comment la sortie a été validée par l'humain

**Sujets à ajouter ou expliciter pour la prochaine itération :**

- limites ou erreurs observées

## 4. Pistes d'action pour la prochaine itération

- Reprendre la requête de la section « Preuve » pour qu'elle s'exécute sur `db/nexamart.duckdb` et qu'elle produise la forme attendue (voir pistes en section 1).
- Compléter `ai-usage.md` en y ajoutant : limites ou erreurs observées.

---

## 5. Traçabilité

- **Run ID :** `20260514T221333Z-7d34bf6a`
- **Devoir :** `S01`
- **Étudiant·e :** `Gia-119`
- **Commit analysé :** `4d22863`
- **Audit (côté instructeur) :** `tools/instructor/feedback_pipeline/audit/20260514T221333Z-7d34bf6a/Gia-119/`
- **Prompts (SHA-256) :**
  - `sql_extractor_system` : `90ee9e277de7a27f...`
  - `rubric_grader_system` : `505f32d1d8319d66...`
  - `ai_usage_grader_system` : `81cb7fdf89bda55a...`
- **Fournisseur (rubrique) :** `openai`
- **Fournisseur (IA-usage) :** `openai` (gpt-5-mini-2025-08-07)

_Ce feedback a été produit par un pipeline automatisé et **revu par l'équipe pédagogique avant publication**. Aucun chiffre ni étiquette de niveau n'est diffusé à ce stade expérimental : l'objectif est uniquement formatif. Ouvrez une issue dans ce dépôt pour toute question._

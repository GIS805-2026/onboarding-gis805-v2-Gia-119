# Rétroaction automatisée -- S01 (Diagnostic fondamental -- NexaMart kickoff)

_Générée le 2026-05-15T12:31:26+00:00 -- Run `20260515T122624Z-00a5a04f`_

Ce document est produit par un pipeline reproductible (vérification SQL déterministe + analyse LLM du brief et de la déclaration IA). Une revue humaine précède toujours sa publication. **À ce stade expérimental, aucune note ni étiquette de niveau n'est diffusée : l'objectif est purement formatif.**

> ⚠️ **Avertissement instructeur (à retirer avant publication) :** cette analyse a été générée avec `--skip-pull`. Le contenu correspond au commit local et **n'est peut-être pas la dernière version poussée par l'étudiant·e**.

---

## 1. Vérification automatique de la requête SQL

La requête extraite de votre brief n'a pas pu être validée automatiquement. Quelques pistes constructives ci-dessous pour vous aider à la rendre exécutable et alignee avec la question posée.

_Observation technique : colonnes manquantes (oracle): quarter_

<details><summary>Requête analysée — cliquez pour déplier</summary>

```sql
WITH revenue_by_month AS ( SELECT p.category, s.region, d.year, d.month, SUM(f.line_total) AS revenue
    FROM raw_fact_sales f JOIN raw_orders o ON f.order_number = o.order_number
    JOIN raw_dim_product p ON f.product_id = p.product_id
    JOIN raw_dim_store s ON f.store_id = s.store_id
    JOIN raw_dim_date d  ON o.order_date = d.date_key
    GROUP BY p.category, s.region, d.year, d.month ), revenue_with_lag AS ( SELECT category, region, year, month, revenue, LAG(revenue) OVER (
            PARTITION BY category, region ORDER BY year, month) AS previous_revenue, revenue - LAG(revenue) OVER (PARTITION BY category, region
            ORDER BY year, month) AS revenue_change
    FROM revenue_by_month)
SELECT * FROM revenue_with_lag WHERE previous_revenue IS NOT NULL ORDER BY revenue_change ASC;
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
> Synonymes acceptés par colonne:
  category: ['category', 'categorie', 'p.category', 'sous_categorie']
  region: ['region', 's.region']
  quarter: ['quarter', 'trimestre', 'd.quarter', 'q']
  revenue: ['net_revenue', 'revenue', 'revenu', 'total_revenue', 'ca', 'sales', 'line_total', 'gross_revenue']

## 2. Rétroaction pédagogique sur le brief

> Le brief présente un bon diagnostic métier et un modèle dimensionnel cohérent (grain et mesures explicites) et inclut une requête de preuve exécutée. Il manque cependant trace du processus (commits, note IA) et des contrôles automatisés reproductibles pour production.

### Observations par dimension

**Model quality**
- Observation : Le brief précise le grain («une ligne de commande (order_number, line_id)»), la mesure calculée (line_total = quantity × unit_price) et les dimensions product/ store.
- Piste d'amélioration : Documenter explicitement le pattern SCD (type 2) et justifier pourquoi/si utilisé, et mentionner les choix de clés substituts vs naturelles.

**Validation quality**
- Observation : Le brief inclut une commande DuckDB et une requête avec LAG pour mesurer l'évolution mensuelle des revenus et affirme avoir exécuté ces contrôles.
- Piste d'amélioration : Ajouter des checks explicites traitant les NULLs, les doublons du grain et un test 'make check' automatique reproduisible.

**Executive justification**
- Observation : La réponse exécutive résume les observations clés («les catégories Toys & Games, Automotive... génèrent les revenus les plus élevés, principalement au Québec et en Ontario») en langage métier.
- Piste d'amélioration : Formuler une recommandation décisionnelle claire (p.ex. prioriser X régions/catégories pour investissement ou réduction de gamme) et ajouter chiffres clés succincts (pct de revenu).

**Process trace**
- Observation : Aucune mention d'un historique de commits git ni d'une note sur l'usage d'IA ou d'un decision log dans le brief.
- Piste d'amélioration : Fournir l'historique git (≥3 commits) avec messages descriptifs et une courte note IA indiquant outils et validation humaine.

**Reproducibility**
- Observation : Le brief donne la commande DuckDB (duckdb db/nexamart.duckdb -c "..."), mais utilise un chemin codé en dur sans README de reproduction.
- Piste d'amélioration : Ajouter un README avec étapes exactes pour cloner, localiser la DB ou loader des données, et un script 'make run' pour exécuter la requête en <5 minutes.

## 3. Déclaration d'utilisation de l'IA

> La déclaration couvre les points demandés (outil nommé, étape d'utilisation et validation humaine), mais l'information sur l'outil reste générique (pas de version/modèle précis) et aucune limite ou erreur rencontrée n'est documentée. Ajoutez la version/modèle exact du service IA utilisé et indiquez les limites ou erreurs observées pour atteindre le score maximal.

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

- **Run ID :** `20260515T122624Z-00a5a04f`
- **Devoir :** `S01`
- **Étudiant·e :** `Gia-119`
- **Commit analysé :** `8224145`
- **Audit (côté instructeur) :** `tools/instructor/feedback_pipeline/audit/20260515T122624Z-00a5a04f/Gia-119/`
- **Prompts (SHA-256) :**
  - `sql_extractor_system` : `90ee9e277de7a27f...`
  - `rubric_grader_system` : `505f32d1d8319d66...`
  - `ai_usage_grader_system` : `81cb7fdf89bda55a...`
- **Fournisseur (rubrique) :** `openai`
- **Fournisseur (IA-usage) :** `openai` (gpt-5-mini-2025-08-07)

_Ce feedback a été produit par un pipeline automatisé et **revu par l'équipe pédagogique avant publication**. Aucun chiffre ni étiquette de niveau n'est diffusé à ce stade expérimental : l'objectif est uniquement formatif. Ouvrez une issue dans ce dépôt pour toute question._

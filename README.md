# off-ai-experiments

Beginner-friendly food search assistant built on Open Food Facts.

It lets users ask natural-language questions like:
- `high protein snack under 200 calories`
- `healthier alternative to nutella`
- `céréales faibles en sucre`

and returns structured, explainable product results.

## What this project does

This project turns a plain user query into:
1. Parsed nutrition intent (constraints/tags/category)
2. Filtered products from Open Food Facts
3. Health insights for each product
4. Better alternatives (comparison mode)

## Data source (API used)

We use **Open Food Facts v2 API**:
- Search endpoint: `https://world.openfoodfacts.org/api/v2/search`
- Product endpoint: `https://world.openfoodfacts.org/api/v2/product/{barcode}.json`

Where in code:
- `src/off_ai/data_adapter.py`

## How data is processed (easy view)

You can think of the system as a pipeline that “segments” raw API data into useful layers:

1. **Input understanding**
    - Detect language (EN/FR)
    - Normalize terms (example: `faibles en sucre` → `low sugar`)
2. **Intent parsing**
    - Extract category, dietary tags, nutrient constraints
    - Example: `under 200 calories` → `energy-kcal_100g <= 200`
3. **API retrieval + filtering**
    - Fetch candidates from Open Food Facts
    - Apply strict nutrient/ingredient filters locally
4. **Smart fallback**
    - If no products are found, relax least-important constraints and retry
5. **Insights + recommendations**
    - Generate risk/positive indicators and health class
    - Optionally suggest better alternatives

## Input → Output (for users)

### Example 1: Search mode

Input:
```bash
python -m off_ai "high protein snack under 200 calories"
```

Output includes:
- Query interpretation (detected language + parsed constraints)
- Top matching products
- Product insights (e.g., Nutri-Score, NOVA, risk indicators)
- If needed: constraint relaxation log

### Example 2: French query

Input:
```bash
python -m off_ai "céréales faibles en sucre"
```

Typical parsed constraint:
- `sugars_100g <= 5.0 g`

### Example 3: Comparison mode

Input:
```bash
python -m off_ai "healthier alternative to nutella"
```

Output includes:
- Reference product details
- Better alternatives with improvement reasons

## Quick start

### 1) Install

```bash
pip install -e .
```

### 2) Run from CLI

```bash
off-ai "low sodium cereal for kids"
off-ai --json "organic high fibre cereal"
python -m off_ai "keto low carb bread"
```

## Python usage

```python
from off_ai import FoodIntelligencePipeline

pipeline = FoodIntelligencePipeline(max_results=10)
result = pipeline.run("high protein vegan snack under 200 calories")
print(result)
```

## Project structure

```text
src/off_ai/
  __init__.py
  __main__.py
  cli.py
  query_preprocessor.py      # EN/FR detection + normalization
  intent_parser.py           # NL query -> FoodQuery
  data_adapter.py            # Open Food Facts API wrapper
  insight_engine.py          # product-level health insights
  recommendation_engine.py   # better alternative ranking
  pipeline.py                # orchestration end-to-end

tests/
  test_query_preprocessor.py
  test_intent_parser.py
  test_data_adapter.py
  test_insight_engine.py
  test_recommendation_engine.py
```

## Running tests

```bash
pytest tests/
```

Current status: all tests passing.

## Notes for beginners

- This project is **rule-based and explainable** (not a black-box ML model).
- API data quality can vary (some products have missing nutrient fields).
- Filtering is strict first, then graceful fallback relaxation if needed.


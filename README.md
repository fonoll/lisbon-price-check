# Lisbon Fair Price — Project Spec (v1)

An interpretable ML tool that estimates a fair market price for an apartment in
Lisbon and tells the user whether a listing is overpriced or underpriced — and why.

**Author:** Anna Egorikhina · **Deadline:** September 1, 2026 · **Status:** spec agreed, data audit in progress

---

## 1. Problem & user

A person (Portuguese local or expat, e.g. Russian-speaking) sees an apartment
listing and wants a second opinion: *is this price fair for this size, district
and condition — and what would a fair price be?*

The tool answers three questions on one screen:
1. **Fair price estimate** (€) for the given parameters
2. **Price per m²** for the apartment vs. the district average
3. **Verdict**: underpriced / fair / overpriced (+ by how many %)

Because the model is interpretable, it also shows *why*: the main factors
(weights) driving the estimate.

## 2. Scope

**v1 (MVP, by Sep 1):**
- One market (sale OR rent — decided by data audit, see §3)
- City of Lisbon, split by district (freguesia)
- Linear regression, implemented from scratch (NumPy) and verified against scikit-learn
- Streamlit web app in English, deployed publicly on Streamlit Community Cloud
- README with metrics, model card and disclaimer

**v2 (after Sep 1, optional):**
- Second market (rent or sale — the other one)
- Fresh data via idealista API (application submitted in week 1)
- Distance to nearest metro station (geocoding + station coordinates)
- District map colored by €/m²; tree-based model as accuracy benchmark

**Out of scope:** user accounts, saved searches, price history/trends,
scraping any portal (violates ToS), any claim of being valuation advice.

## 3. Data strategy

| Layer | Source | Role |
|---|---|---|
| Training data (v1) | Kaggle: Portuguese real-estate dataset + Lisbon House Prices | Immediate start; pipeline is source-agnostic |
| Training data (v2) | idealista official API (apply day 1; approval takes time) | Fresh listings, real product |
| Sanity check | INE (Statistics Portugal) €/m² aggregates by freguesia | Validate model's district averages against official stats |

**Week-1 decision point — sale vs rent:** download both candidate datasets,
compare row count, share of missing values, and whether freguesia/district is
present. Launch v1 on whichever market has the bigger & cleaner dataset.

## 4. Features

| Tier | Features | Rule |
|---|---|---|
| 1 — must have | area (m²), district (freguesia), rooms (T0/T1/T2/T3+) | v1 blocked without these |
| 2 — if present in data | floor, condition (new/used/needs renovation), year built, elevator, furnished (rent) | include when column exists & <30% missing |
| 3 — derived, v2 | distance to metro/transport | needs geocoding; not in v1 |

Rare districts with too few listings are grouped into "Other" to keep
weights stable.

## 5. Model

- **v1 model:** multiple linear regression. Numeric features standardized;
  district and condition one-hot encoded. Trained twice: own NumPy gradient
  descent (learning goal) and scikit-learn (verification — coefficients must match).
- **Interpretability requirement:** the app shows the top factors and their
  effect in € (e.g. "each m² in Arroios ≈ +X €", "needs renovation ≈ −Y €").
- **Evaluation:** train/test split (80/20), report MAE, RMSE, R², and MAPE.
  Baseline to beat: predicting the district average €/m² × area.
- **Fair-price verdict (proposal — to confirm):**
  - within ±10% of prediction → "fair price"
  - 10–25% above → "overpriced", >25% → "strongly overpriced"
  - symmetric for underpriced
- Disclaimer in app and README: educational project, not valuation advice.

## 6. App (Streamlit, English)

Single screen: input form (tier-1 + available tier-2 features + asking price) →
output card with fair price, €/m² vs district average, verdict badge,
and "what drives this estimate" factor list. Deployed on Streamlit Community
Cloud; link in README and LinkedIn project entry.

## 7. Success criteria (Sep 1)

1. Public URL works; a stranger can get a verdict in under a minute
2. Test-set MAPE reported in README (target: beat the district-average baseline; stretch: MAPE ≤ 20%)
3. README contains: data source & date, features, metrics, weights table, limitations
4. Own gradient-descent implementation matches sklearn coefficients
5. Project entry added to LinkedIn with the live link

## 8. Timeline (8 weeks, ~1h/day + Saturday deep-work block)

| Week | Milestone |
|---|---|
| 1 (Jul 7–13) | Repo created; idealista API application sent; both datasets audited; sale-vs-rent decision |
| 2 | Cleaning + EDA: distributions, €/m² by district, missing values handled |
| 3 | v0 model: area-only regression, own gradient descent vs sklearn |
| 4 | Full linear model: one-hot districts, scaling, train/test, metrics |
| 5 | Fair-price logic + weights interpretation; beat the baseline |
| 6 | Streamlit app locally: form → estimate → verdict |
| 7 | Deploy, polish UI, write README/model card |
| 8 (by Sep 1) | Buffer for surprises; LinkedIn project entry + optional post |

## 9. Risks

| Risk | Mitigation |
|---|---|
| idealista API not approved in time | v1 ships on Kaggle data; API becomes v2 |
| Kaggle data outdated (~2020) | State data date visibly; INE aggregates as reality check; disclaimer |
| Too few listings per district | Group rare districts into "Other" |
| Semester starts, time drops | Weeks 1–6 do the heavy lifting; week 8 is buffer |

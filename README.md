# BeautySeq Recommender Web Demo

This project demonstrates an e-commerce product recommendation system on the Amazon Beauty dataset. The web demo uses `Popularity` for cold-start users and switches to `BSARec` once the user has enough product interactions.

## Goals

- Display a Beauty product catalog.
- Track user behavior such as product views and add-to-cart actions.
- Generate personalized recommendations from the user's recent behavior sequence.
- Report offline evaluation results for `Popularity`, `GRU4Rec`, `SASRec`, `BERT4Rec`, and `BSARec`.

## How To Run

Install dependencies:

```powershell
pip install -r requirements.txt
```

Start the web demo:

```powershell
cd "C:\Study\DoAnV2\Web demo"
python server.py
```

Open:

```text
http://127.0.0.1:8501
```

Default demo account:

```text
demo / demo123
```

You can also register a new account in the UI. For presentation, a new account is recommended because it gives a clean behavior sequence.

## Recommendation Flow

The system has two recommendation modes:

1. `Popularity`

   Used when the user is not logged in or has fewer than 3 product behavior events. Products are ranked by interaction frequency in `train_history.csv`.

2. `BSARec`

   Used from the 3rd product behavior event onward. The server reads the user's recent item sequence, feeds it into the BSARec checkpoint, scores candidate items, removes already-seen/cart items, and returns the top-ranked products.

Search and category filtering are catalog filtering features. They are not recommendation models.

## User Behavior Logs

Demo behavior is stored in:

```text
Web demo/demo_events.jsonl
```

Each line is a JSON event:

```json
{"event_id":"evt_xxx","user_id":"demo_xxx","type":"view_product","item_id":"594","query":null,"created_at":"2026-06-01T01:02:31.856002+00:00"}
```

Events used as BSARec behavior signals:

```text
view_product
add_to_cart
```

Other events may be logged but are not used in the BSARec sequence:

```text
search
open_cart
remove_from_cart
```

## How BSARec Generates Recommendations

The recommendation logic is implemented in `Web demo/server.py`:

1. Read the user's events from `demo_events.jsonl`.
2. Keep only `view_product` and `add_to_cart` item events.
3. Keep up to the 50 most recent item IDs.
4. Use `Popularity` if the sequence has fewer than 3 events.
5. Use `BSARec` if the sequence has at least 3 events.
6. Score all items in the BSARec item embedding space.
7. Exclude items already viewed or already in the cart.
8. Return the top-K items to the web UI.

## Important Files

```text
Web demo/server.py                         Backend API and recommendation logic
Web demo/app.js                            Frontend logic
Web demo/index.html                        Web UI
Web demo/styles.css                        UI styles
Web demo/items.csv                         Basic product metadata
Web demo/item_details.csv                  Store, category, description, price
Web demo/item_images.csv                   Product image URLs
Web demo/train_history.csv                 Training interaction history for popularity
Web demo/metrics.csv                       Offline evaluation results
Web demo/demo_users.json                   Demo user accounts
Web demo/demo_carts.json                   Demo cart state
Web demo/demo_events.jsonl                 Demo behavior logs
Web demo/beauty_recommender_outputs/       Model/evaluation artifacts
BSARec-main/src/output/BSARec_Beauty_best.pt  BSARec checkpoint
```

## Evaluation Results

Main results are stored in `Web demo/metrics.csv`.

| Model | HR@10 | NDCG@10 | MRR |
|---|---:|---:|---:|
| Popularity | 0.011984 | 0.005613 | 0.005655 |
| GRU4Rec | 0.030631 | 0.014568 | 0.014301 |
| SASRec | 0.048607 | 0.025990 | 0.024557 |
| BERT4Rec | 0.075661 | 0.042717 | 0.039234 |
| BSARec | 0.095023 | 0.057535 | 0.052632 |

BSARec achieves the best result across HR@K, NDCG@K, and MRR, showing that it learns useful sequential behavior signals beyond simple popularity.

## Suggested Demo Scenario

Use a new account, then click products from a consistent theme. Example skin care/body care sequence:

```text
1089  Liquid Trust
518   Estee Lauder Perfumed Body Powder
1284  Bikini Hair Removal System + Shave Cream
1111  Diva By Emanuel Ungaro Body Lotion
62    Witch Vera Gel
```

After clicking these products, return to the home page. If the model banner shows `BSARec`, explain:

```text
The user has viewed several skin care, body care, and grooming products.
BSARec uses this behavior sequence to re-rank candidate products and recommend items that are likely to be relevant next.
```

## Notes For Presentation

- BSARec is not a keyword filter or a category filter.
- It learns sequential patterns: after a user interacts with items A, B, and C, which item is likely to come next.
- Some recommendations may look noisy because the Amazon Beauty catalog contains mixed categories and imperfect product titles.
- For a clearer demo, use a new account and click products from the same theme before showing the recommendation page.

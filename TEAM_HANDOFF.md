# TEAM HANDOFF — Amazon ML Challenge 2026


Ye document team ke dono members ke liye hai. Isme current progress, problem, aur har member ka kaam diya gaya hai & Kaam start karne se pehle is file ko poora read kar lena.

& Common resources ye sab hai ok:

Complete dataset, official problem statement PDF, official guidelines PDF aur baaki resources Drive folder mein dete jaayenge

Drive:
https://drive.google.com/drive/folders/1zJh76qw0HZB2LIG5m_7LySk7A6-Jgr-H?usp=drive_link


Member 1 — , Matching Model > 
> 2 PM tak first usable matching model + validation F0.5 + threshold + prediction code de do.

Tumhara kaam candidate pairs ke upar matching model banana hai &

candidate_pairs_stage1c.parquet ye current blocking stage ka output hai & is file mein possible S1 → S2/S3 candidate pairs hain. Is file ko matching model ke input ke roop mein use karna hai aur Blocking dobara karne ki zarurat nahi hai.

++ Is file ko modify nahi karna hai.**

and Use candidate_pairs_stage1c.parquet ke saath train_source1.tsv, train_source2.tsv, train_source3.tsv aur train_ground_truth.tsv.

so Tumhara goal pair features banana, positive aur hard-negative pairs prepare karna, matching model train karna aur Macro F0.5 ke according threshold tune karna hai.

aur ek baat Important: ek S1 entity ke zero, one ya multiple correct matches ho sakte hain. so Sirf top-1 match assume nahi karna hai.


Member 2 — < Blocking Improvement >

> 1:30 PM tak improved blocking strategy + recall + candidate count + runtime de do. Focus recall improve karne par hai, candidate explosion nahi.

Tumhara kaam current blocking strategy ko improve karna hai.....

mtlb Current blocker ka recall lagbhag 60% hai. Missed GT links mein se bahut saare existing blocking keys share karte hain, lekin bade blocks ko current caps ki wajah se skip kiya ja raha hai.

Tumhe better blocking strategy test karni hai, especially composite ya alternative blocking methods, aur recall ko improve karna hai bina candidate pairs ko bahut zyada badhaye....//

aur Har experiment ke liye blocking recall, candidate count aur runtime note karna hai.****    .................. aur Candidate file banane ke alawa submission/output files ke liye validation code bhi ready karna hai.


aur ek Important chiz:

candidate_pairs_stage1c.parquet sirf Member 1 ke matching-model work ke liye current benchmark candidate set hai.

Member 2 ka main kaam new blocking strategy banana hai. Isliye woh current parquet ko final candidate set nahi samjhega.

Final candidate set baad mein improved blocking ke basis par generate hoga.

mai final integration karega aur dono members ke kaam ko combine karega.

> 2 PM ke baad main dono outputs integrate karke final pipeline, test prediction aur submission handle karunga.

====
====
====


## 1. as u know ki What is the problem?

so We have to solve **Business Entity Resolution**.

where there are 3 data sources:

- **Source 1 (S1):** reference entities
- **Source 2 (S2):** noisy business records
- **Source 3 (S3):** noisy business records

& For each S1 entity, the correct answer can be:

- **no match**
- **one match**
- **multiple matches**

The final matching result must use only IDs that are present in the candidate set.

---

## 2. Dataset

Current training data sizes:

| File | Rows |
|---|---:|
| S1 | 2,206,821 |
| S2 | 5,034,616 |
| S3 | 5,285,603 |

Ground-truth statistics:

| Type | Count |
|---|---:|
| S1 with no match | 123,247 |
| S1 with exactly 1 match | 119,157 |
| S1 with multiple matches | 1,964,417 |
| Maximum matches for one S1 | 11 |

**Important:** Do NOT assume “one S1 = one best record”.

---

## 3. Current experiment

We are using **100,000 S1 entities** as a benchmark.

Current blocking methods:

- `key_name2`
- `key_name1`
- `key_postal`

Current caps:

- `key_name2` → 100
- `key_name1` → 30
- `key_postal` → 30

Current candidate set:

**4,720,098 candidate pairs**

Average:

**47.2 candidates per S1**

---

## 4. Current blocking result

True GT links in the 100k benchmark:

**346,089**

Captured by current blocker:

**207,588**

Blocking recall:

**59.9811%**

Missed true links:

**138,501**

So our biggest problem right now is **candidate generation / blocking**, not the final matcher.

---

## 5. What we discovered

We checked the missed GT links.

For missed links:

### S2

- 88.44% share `key_postal`
- 41.52% share `key_name1`
- 7.62% share `key_name2`
- 95.18% have at least one shared blocking key, but all shared blocks are larger than the current caps

### S3

- 89.04% share `key_postal`
- 41.59% share `key_name1`
- 6.97% share `key_name2`
- 95.46% have at least one shared blocking key, but all shared blocks are larger than the current caps

### Important conclusion

The existing blocking keys are **not useless**.

A large part of the recall loss happens because the corresponding blocks are too large and are being skipped by the current caps.

However, **do not simply increase `key_postal` cap**.

Some postal blocks are extremely large:

- S2: up to about **1.83 million rows**
- S3: up to about **1.93 million rows**

So a large postal cap can create a huge number of candidate pairs.

---

## 6. Composite blocking experiments

We tested:

- `country + key_name2`
- `country + key_name1`
- `country + key_postal`
- `country + key_name1 + key_postal`
- `country + key_name2 + key_postal`

The most promising combinations were:

### `country + key_name2 + key_postal`

Among missed links that share this block:

- S2: 74.60% are within a block size of 500
- S3: 73.22% are within a block size of 500

### `country + key_name1 + key_postal`

Among missed links that share this block:

- S2: 62.32% are within a block size of 500
- S3: 59.70% are within a block size of 500

These are **diagnostic results**, not final settings.

---

<<<<<<<< ye niche k aportion samjhane keliye banaaye hai gpt se so agar kux smjh naa aye to pux lena>>>>>>>>>>>>>>>>>>

````md
# 7. Team Tasks

## Member 1 — Matching Model

### Goal

Build the matching model using the candidate pairs already generated by the team.

The file:

`candidate_pairs_stage1c.parquet`

is the current candidate set generated by our blocking stage for the 100,000 S1 benchmark.

Since you do not have access to the Team Lead's Kaggle notebook, download this parquet file from the shared resources and upload it to your own Kaggle notebook/environment.

Use it together with:

- train_source1.tsv
- train_source2.tsv
- train_source3.tsv
- ground_truth.tsv

### Work on

- Create pair-level features such as business name similarity, address similarity, token overlap, postal match and country match.
- Prepare positive and hard-negative training pairs.
- Train a practical matching model.
- Tune the decision threshold using Macro F0.5.
- Support zero, one, or multiple matches for one S1 entity.

### Deliver to Team Lead

- training code
- prediction code
- model file
- best threshold
- validation Macro F0.5
- short explanation of features used

---

## Member 2 — Blocking Improvement

### Goal

Improve the current blocking strategy and increase blocking recall without creating too many candidate pairs.

You do not have access to the Team Lead's Kaggle notebook, so run your experiments in your own Kaggle notebook/environment using the provided dataset and ground truth.

Use the current blocking results in this document as the baseline.

### Work on

Try improved blocking methods such as:

- composite blocking
- name + address combinations
- house-number + postal combinations
- character n-gram based blocking
- other efficient exact or near-exact blocking keys

For every experiment record:

```text
method
candidate count
blocking recall
average candidates per S1
runtime
````

### Deliver to Team Lead

* blocking code
* best blocking strategy
* blocking recall
* candidate count
* runtime
* short explanation of the improvement

The Team Lead will later integrate the selected blocking strategy into the final pipeline.

---

# 8. Team Lead — Integration

Current owner: **Shreyansh**

I will review both members' results and combine the best parts of both works into the final solution.

My responsibilities are:

* compare blocking experiments
* select and integrate the final blocking strategy
* integrate Member 1's matching model
* perform final threshold tuning
* run the complete pipeline on the test data
* generate `candidate_pairs.tsv`
* generate `matching_results.tsv`
* validate the final submission
* prepare the final README and documentation

---

# 9. Important Rules for Everyone

### Do not

* assume top-1 matching is enough
* blindly increase large block caps
* use external business databases or internet-based business data
* treat the current 59.98% blocking recall as final
* treat the current blocking caps as official requirements

### Do

* keep experiments reproducible
* report recall and candidate count together
* use hard negatives where possible for model training
* test changes on the same benchmark when possible
* keep your code ready for final integration

---

# 10. Current Priority

1. Member 1: build and validate the matching model.
2. Member 2: improve blocking recall without candidate explosion.
3. Team Lead: combine both results and build the final end-to-end pipeline.



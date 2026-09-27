# TEAM HANDOFF — Amazon ML Challenge 2026

# TEAM HANDOFF

Ye document team ke dono members ke liye hai. Isme current progress, problem, aur har member ka kaam diya gaya hai. Kaam start karne se pehle is file ko poora read kar lena.

Common resources:

Complete dataset, official problem statement PDF, official guidelines PDF aur baaki resources Drive folder mein diye gaye hain.

Drive:
https://drive.google.com/drive/folders/1zJh76qw0HZB2LIG5m_7LySk7A6-Jgr-H?usp=drive_link


Member 1 — Matching Model

Tumhara kaam candidate pairs ke upar matching model banana hai.

candidate_pairs_stage1c.parquet current blocking stage ka output hai. Is file mein possible S1 → S2/S3 candidate pairs hain. Is file ko matching model ke input ke roop mein use karna hai. Blocking dobara karne ki zarurat nahi hai.

Is file ko modify nahi karna hai.

Use candidate_pairs_stage1c.parquet ke saath train_source1.tsv, train_source2.tsv, train_source3.tsv aur train_ground_truth.tsv.

Tumhara goal pair features banana, positive aur hard-negative pairs prepare karna, matching model train karna aur Macro F0.5 ke according threshold tune karna hai.

Important: ek S1 entity ke zero, one ya multiple correct matches ho sakte hain. Sirf top-1 match assume nahi karna hai.


Member 2 — Blocking Improvement

Tumhara kaam current blocking strategy ko improve karna hai.

Current blocker ka recall lagbhag 60% hai. Missed GT links mein se bahut saare existing blocking keys share karte hain, lekin bade blocks ko current caps ki wajah se skip kiya ja raha hai.

Tumhe better blocking strategy test karni hai, especially composite ya alternative blocking methods, aur recall ko improve karna hai bina candidate pairs ko bahut zyada badhaye.

Har experiment ke liye blocking recall, candidate count aur runtime note karna hai.

Candidate file banane ke alawa submission/output files ke liye validation code bhi ready karna hai.


Important:

candidate_pairs_stage1c.parquet sirf Member 1 ke matching-model work ke liye current benchmark candidate set hai.

Member 2 ka main kaam new blocking strategy banana hai. Isliye woh current parquet ko final candidate set nahi samjhega.

Final candidate set baad mein improved blocking ke basis par generate hoga.

Shreyansh final integration karega aur dono members ke kaam ko combine karega.

====
====
====


## 1. What is the problem?

We have to solve **Business Entity Resolution**.

There are 3 data sources:

- **Source 1 (S1):** reference entities
- **Source 2 (S2):** noisy business records
- **Source 3 (S3):** noisy business records

For each S1 entity, the correct answer can be:

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

# 7. Team Tasks

## Member 1 — Matching Model

### Goal

Build the **pair matching model** using the current candidate pairs.

Use:

`candidate_pairs_stage1c.parquet`

Path in Kaggle:

```text
/kaggle/working/candidate_pairs_stage1c.parquet
```

Size:

about **44.9 MB**

### Work on

1. Join candidate pairs with S1/S2/S3 attributes.
2. Create pair-level features, for example:
   - business name similarity
   - address similarity
   - token overlap
   - postal exact match
   - country match
   - length differences
3. Create positive and hard-negative training pairs.
4. Train a practical pair classifier.
5. Tune the decision threshold using the actual **Macro F0.5** metric.
6. Make sure one S1 can return:
   - zero matches
   - one match
   - multiple matches

### Deliver to Team Lead

- model/training code
- prediction code
- best threshold
- validation Macro F0.5
- short explanation of features

---

## Member 2 — Alternate Blocking + Output

### Goal A: Improve blocking recall

Develop an **independent blocking strategy**.

Try ideas such as:

- composite blocking
- character n-gram blocking
- name + address components
- house-number + postal combinations
- other low-cost exact/near-exact blocking keys

Main target:

> Recover missed GT links without causing a huge candidate explosion.

For every experiment report:

```text
method
candidate count
blocking recall
average candidates/S1
runtime
```

### Goal B: Submission/output code

Prepare code for:

```text
candidate_pairs.tsv
matching_results.tsv
```

Make sure:

- one row exists for every test S1
- empty candidate lists are allowed
- no duplicate candidate IDs
- predicted match IDs are a subset of candidate IDs
- only valid S2/S3 IDs are used

### Deliver to Team Lead

- blocking code
- recall result
- candidate count
- runtime
- final output/validator helper code

---

# 8. Team Lead — Integration

Current owner: **Shreyansh**

Responsibilities:

- compare blocking experiments
- combine the strongest safe blocking methods
- validate blocking recall
- integrate Member 1's matcher
- tune final threshold
- generate final candidate and matching files
- validate final submission structure
- prepare README/documentation

---

# 9. Important Rules for Everyone

### Do not

- assume top-1 matching is enough
- blindly increase postal block caps
- use external business databases / internet augmentation
- treat the current 59.98% blocking recall as final
- treat current caps as official challenge requirements

### Do

- measure every blocking change on the same 100k benchmark
- record candidate count and recall together
- prefer hard negatives over random easy negatives for matcher training
- keep code reproducible
- keep experiments separate so results can be compared

---

# 10. Current Priority

## Priority 1
Improve blocking recall without exploding candidate count.

## Priority 2
Build and validate the matching model in parallel.

## Priority 3
Integrate both and generate valid final submission files.

---

## One-line summary

**Our current blocker reduces the search space extremely well, but recall is only ~60%. Most missed true links already share a blocking key; they are mainly being lost because large blocks are being skipped. The next goal is smarter composite blocking, not blindly larger caps.**

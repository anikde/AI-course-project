CSL7610 Artificial Intelligence - Assignment
Classical Reasoning System for Logical Entailment from Natural Language

Anik De (P25CS0001), PhD


FILES
-----
reasoning_agent.ipynb   complete implementation, experiments, figures, reasoning
                        traces; prints the final accuracy and inference time and
                        writes submission.csv
report.pdf              representation, algorithms, experiments, reasoning traces
submission.csv          predictions, columns: id,label  (label = True / False)
README.txt              this file


REQUIREMENTS
------------
Python 3.10 or newer (developed with Python 3.14).

Needed to predict on a test file:
    pip install pandas spacy
    python -m spacy download en_core_web_sm

Needed only for the experiments on the public data (RUN_EXPERIMENTS = True):
    pip install datasets huggingface_hub matplotlib numpy

The notebook does not install anything itself. If spaCy or its model is
missing, it says so; RuleTaker-style text still works with a built-in
fallback lemmatiser, but free-form English (FOLIO style) needs spaCy.
No Hugging Face login is needed: FOLIO is read from the authors' public
GitHub copy (v0.0) and cached in ~/.cache/reasoning_agent/.


HOW TO RUN
----------
1. Open reasoning_agent.ipynb and edit CONFIG in the first code cell:

       'TEST_PATH': 'test.csv',      # path of the test file
       'RUN_EXPERIMENTS': True,      # False = fast run, test file only

2. Run all cells from a fresh kernel (Kernel > Restart & Run All), or:

       jupyter nbconvert --to notebook --execute reasoning_agent.ipynb --inplace

   RUN_EXPERIMENTS = True   loads the public datasets, runs every experiment
                            and ablation, draws the figures, then predicts.
                            About 1 minute (plus the first-time download).
   RUN_EXPERIMENTS = False  only loads TEST_PATH, predicts, writes
                            submission.csv and prints accuracy (if the file
                            has labels) and inference time. A 3,504-row test
                            file takes about 3 seconds in total.

   If TEST_PATH does not exist and RUN_EXPERIMENTS = True, the predictions
   are made for the public evaluation set instead (this is how the included
   submission.csv was produced). If TEST_PATH does not exist and
   RUN_EXPERIMENTS = False, the notebook stops with a clear message.

3. The last cells print, for example:

       FINAL ACCURACY      : 99.94%
       TOTAL INFERENCE TIME: 3.60 s  (0.024 ms/query)


TEST FILE FORMAT
----------------
CSV, TSV, JSON, JSONL or Parquet, one row per query. Column names are matched
ignoring case, spaces, '-' and '_':

  id         id, example_id, question_id, qid, idx, uid, item_id, index
             (if absent, the row number is used)
  knowledge  context, premises, theory, kb, knowledge_base, facts, facts_rules,
  base       given_info, story, passage, rules, background, input
  statement  statement, question, query, hypothesis, conclusion, claim, goal
  label      label, flag, answer, gold, target, truth, gold_label, y  (optional)
  logic      premises_fol / conclusion_fol (optional; used when present)

A knowledge base may be one text or a list of sentences. Labels count as True
when they are true / entailment / yes / 1; everything else (False, Uncertain,
Unknown, not entailment, no) counts as not entailed. If the knowledge-base or
statement column cannot be found, the notebook stops and lists the columns it
saw; add the name to ALIASES in the notebook (section 3.1) to fix it.


METHOD (details in report.pdf)
------------------------------
Natural language -> internal representation -> classical reasoning -> True/False.
No generative model is used; spaCy is used only for lemmas and dependency parses.

  Facts and if-then rules     Horn grammar -> stratified forward chaining with
  (RuleTaker style)           unification and negation as failure (closed world)
  General first-order text    dependency-based NL->FOL translator (or a formula
  (FOLIO style)               parser when logic is given) -> CNF -> resolution
                              refutation (set of support, unit preference,
                              factoring, 2,000-step budget; open world)
  Causal text, counterfactual causal parser -> Boolean structural causal model
  questions (CounterBench)    -> abduction, intervention (do), prediction

Backward chaining is implemented as well, but only for comparison in the
experiments; the system's router uses forward chaining.


RESULTS ON THE PUBLIC DATA (from the submitted notebook run)
------------------------------------------------------------
  RuleTaker test (151,911 queries)              99.97 %   3.40 s
  FOLIO validation, English input (204)         66.18 %   1.28 s
  FOLIO validation, gold FOL input (204)        98.04 %   0.32 s
  CounterBench held-out half (600)              94.00 %   0.05 s
  Final evaluation set (152,715)                99.94 %   3.60 s

Design decisions were made on RuleTaker dev, FOLIO train and the CounterBench
dev half; the numbers above come from the other splits. Results are
deterministic (fixed seed, step budget instead of a time limit). Timing is on
an Intel Core i5-12400, and covers parsing and reasoning for every query, with
everything the system remembers cleared before each timed run; data loading is
excluded.


NOTES
-----
- Figures and results.json from the experiments are saved to
  ~/.cache/reasoning_agent/ (CONFIG['FIG_DIR']), not to this folder.
- Knowledge bases fully covered by the Horn grammar are read under the
  closed-world assumption (RuleTaker convention); everything else under the
  open-world assumption.
- Free-form English (FOLIO style) is the weakest input type: 66.18 % against a
  64.71 % always-False baseline. See the limitations section of report.pdf.

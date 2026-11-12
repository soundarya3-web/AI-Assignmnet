REASONING AGENT
===============
Decides whether a query follows from a knowledge base.
Answer: True (the query is entailed) or False (it is not).
If the solver does not settle the query, the stored prediction is Unresolved.
Unresolved is not treated as False.


FILES
-----
reasoning_agent.ipynb   the notebook. It scores a logic test file, and, when asked,
                        scores the public datasets and rewrites submission.csv
submission.csv          predictions on the public datasets
                        columns: dataset, sample_no, prediction
                        4840 rows: RuleTaker 1625, CounterBench 1200,
                        FOLIO with Uncertain removed 811,
                        FOLIO with Uncertain counted as False 1204
submission_test.csv     created when the notebook is run on a test file (not included)
report.pdf              description of the method and the public-dataset results
README.txt              this file

The notebook imports these modules from the project folder that contains it
(the parent of this submission folder):
    fol_engine.py            Z3 entailment
    fol_engine_baseline.py   parser used by the Z3 engine
    nl_to_fol_pipeline.py    RuleTaker and CounterBench translation, forward chaining
    resolution_engine.py     fast resolution
    datasets_unified/        CounterBench and the binary FOLIO logic file
    datasets_local/          RuleTaker splits and the original FOLIO jsonl files


REQUIREMENTS
------------
Python 3.9 or newer
    pip install pandas matplotlib z3-solver

The public datasets used below are already stored in datasets_unified and
datasets_local. Reproducing the reported results does not need a Hugging Face
login.


HOW TO RUN A TEST FILE
----------------------
The test file contains logic directly. No English translation is used.

1. Put the test file in the same folder as the notebook.
2. In the Configuration cell set:
       INPUT_CSV = "hidden_test.csv"     (name of the test file)
       RECHECK_RESULTS = False
3. Run all cells.
   The hidden-test cell prints the number of examples, the total inference time,
   the accuracy (if the file has labels), and the TP / FN / FP / TN counts,
   and writes the predictions to submission_test.csv.

The decision is made by predict_logic(context, statement). It sends the formulas
to Z3. If the premises together with the negation of the statement are
unsatisfiable, the statement follows and the answer is True. If they are
satisfiable, the answer is False. If Z3 returns unknown, or the formula does not
parse, the prediction stays Unresolved.


INPUT FORMAT
------------
A CSV file with the columns:
    id          example id
    context     the knowledge base, one formula per line
    statement   the query (one formula)
    label       optional: True / False (anything else, e.g. Uncertain, counts as False)

Example row:
    id:        1
    context:   Young(Bob)
               Kind(Bob)
               ∀x (Young(x) → Smart(x))
               ∀x (Smart(x) ∧ Kind(x) → Successful(x))
    statement: Successful(Bob)
    label:     True

submission_test.csv has the columns id and flag.
flag is True, False, or Unresolved.


LOGIC SYNTAX
------------
    not                ¬     ~
    and                ∧     &
    or                 ∨     |
    exclusive or       ⊕     xor
    implies            →     ->
    if and only if     ↔     <->
    for all            ∀x    forall x
    there exists       ∃x    exists x
    equality           =     ≠

Predicates are written Name(arg1, arg2). A single name such as P is a proposition.
Quantified names are variables. Distinct constants are treated as different names
when the formulas contain no positive equality.


REPRODUCING THE REPORTED RESULTS
--------------------------------
1. pip install pandas matplotlib z3-solver
2. Set RECHECK_RESULTS = True in the Configuration cell and run all cells.
   This scores the local public files and writes submission.csv.
3. Set RECHECK_RESULTS = False again afterwards.

What runs when RECHECK_RESULTS is True:

    RuleTaker
        English facts and rules from datasets_local/ruletaker_train.csv,
        ruletaker_test.csv, and ruletaker_ood_test.csv.
        The text is turned into ground Horn clauses and tested by forward chaining.
        1625 examples. Operational accuracy 98.46% (1600/1625).
        Reasoning time 0.000013 ± 0.000003 s/sample.
        Splits: train 98.30% (983/1000), test 97.60% (244/250),
        OOD 99.47% (373/375).

    CounterBench
        English stories from datasets_unified/counterbench_unified.csv.
        Each story becomes a twin network: a factual world and a counterfactual
        world. An intervention deletes the structural equation of the intervened
        event and sets that event false in the counterfactual world.
        Fast resolution tests the query (predicate index, unit subsumption,
        and unit simplification). Same proof rules as plain resolution.
        1200 examples. Operational accuracy 94.58% (1135/1200).
        Reasoning time 0.0022 ± 0.0048 s/sample.

    FOLIO, Uncertain removed
        The supplied formulas in datasets_unified/folio_fol_unified.csv.
        Z3, with the same call as the hidden-test inference, and with predicate
        alignment on because some FOLIO predicates are spelled in more than one way.
        811 examples. Operational accuracy 76.82% (623/811).
        Evaluated accuracy 77.01% (623/809). Coverage 99.75%.
        Two queries stayed unknown and are stored as Unresolved.
        Reasoning time 0.0236 ± 0.0597 s/sample.

    FOLIO, Uncertain counted as False
        datasets_local/folio_train.jsonl and folio_validation.jsonl.
        The formulas are used as given. The source files are not rewritten.
        Entailment and True become True. Contradiction, False, and Uncertain
        become False. A solver unknown stays Unresolved.
        1204 examples. Operational accuracy 82.48% (993/1204).
        Evaluated accuracy 82.68% (993/1201). Coverage 99.75%.
        Three queries stayed unknown.
        Reasoning time 0.0297 ± 0.0620 s/sample.
        The 811 binary examples inside this run again score 76.82%.

Accuracy is the same on every run. The times change slightly from run to run
and depend on the machine. These times were measured on Windows, Python 3.10,
one CPU thread. The comparison is also in report.pdf.


DATASETS
--------
RuleTaker     https://huggingface.co/datasets/jise/ruletaker
              local files: datasets_local/ruletaker_train.csv
                           datasets_local/ruletaker_test.csv
                           datasets_local/ruletaker_ood_test.csv
CounterBench  https://huggingface.co/datasets/CounterBench/CounterBench
              local file:  datasets_unified/counterbench_unified.csv
FOLIO         https://huggingface.co/datasets/yale-nlp/FOLIO
              local files: datasets_unified/folio_fol_unified.csv
                           datasets_local/folio_train.jsonl
                           datasets_local/folio_validation.jsonl

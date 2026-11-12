# Classical Entailment for RuleTaker, CounterBench, and FOLIO

**Author:** Jatoth Soundarya  
**Roll No.:** M24MA2004  
**Programme:** M.Tech DCS

## 1\. Project Overview

This project implements a reasoning system that checks whether a query follows from a given knowledge base. The knowledge base contains facts and rules, and the system decides whether the query is logically entailed by them.

The project uses three public datasets: RuleTaker, CounterBench, and FOLIO. Since the datasets represent logical reasoning problems in different ways, the implementation uses a suitable reasoning procedure for each dataset.

The system returns **True** when the premises entail the query and **False** when they do not. If a solver cannot reach a decision within its limits, the result is kept as **Unresolved** rather than being treated as False.

## 2\. Datasets

### RuleTaker

RuleTaker contains short English stories made up of facts and if-then rules. In this project, the released train, test, and out-of-distribution test splits are used.

* Total examples: 1,625
* Train: 1,000
* Test: 250
* OOD test: 375
* Dataset: [RuleTaker on Hugging Face](https://huggingface.co/datasets/jise/ruletaker)

### CounterBench

CounterBench contains counterfactual questions based on fictional causal stories. The questions ask whether an outcome would still happen after an intervention, such as switching off a particular event.

* Total examples: 1,200
* Dataset: [CounterBench on Hugging Face](https://huggingface.co/datasets/CounterBench/CounterBench)

### FOLIO

FOLIO contains natural-language stories paired with first-order logic premises and conclusions. The annotated logical formulas supplied with the dataset are used directly.

* Train: 1,001
* Validation: 203
* Total: 1,204
* Labels: True, False, and Uncertain
* Dataset: [FOLIO on Hugging Face](https://huggingface.co/datasets/yale-nlp/FOLIO)

## 3\. Methodology

The implementation follows a separate reasoning pipeline for each dataset.

### 3.1 RuleTaker: Forward Chaining

The English facts and rules are translated into ground Horn clauses. Forward chaining starts with the facts and repeatedly applies rules whenever their conditions are satisfied. The process stops when no new facts can be derived.

The query is classified as True if it is present in the final closure. Otherwise, it is classified as False. This procedure computes the complete closure for the ground Horn-rule fragment used here.

### 3.2 CounterBench: Twin Network and Fast Resolution

For counterfactual reasoning, the implementation keeps the factual world and the intervened world separate using a twin-network representation.

The causal statements are represented as logical formulas. The intervention changes the selected event in the counterfactual world, while the remaining causal relationships are retained according to the model described in the report. The resulting formulas are converted into clauses and checked using resolution refutation.

The fast resolution procedure uses predicate indexing, unit subsumption, and unit simplification to reduce unnecessary clause processing. A time limit of 0.3 seconds is used per query. If the search stops before reaching a decision, the result remains Unresolved.

### 3.3 FOLIO: Z3 Satisfiability Checking

FOLIO already provides first-order logic formulas, so the annotated premises and conclusion are parsed and passed to the Z3 satisfiability solver.

The solver checks the premises together with the negation of the query:

* If the resulting formulas are unsatisfiable, the query is entailed and the prediction is True.
* If they are satisfiable, the query is not entailed and the prediction is False.
* If Z3 returns unknown or the one-second limit is reached, the prediction is Unresolved.

Two evaluation settings are considered for FOLIO:

1. Remove the Uncertain examples and evaluate only the True and False labels.
2. Keep all examples and count Uncertain as False for the binary evaluation.

## 4\. Results

The reported operational accuracy counts an Unresolved prediction as incorrect.

|Dataset|Method|Examples|Correct|Accuracy|
|-|-|-:|-:|-:|
|RuleTaker|Forward chaining|1,625|1,600|98.46%|
|CounterBench|Fast resolution|1,200|1,135|94.58%|
|FOLIO (Uncertain removed)|Z3|811|623|76.82%|
|**Combined**|**Dataset-specific methods**|**3,636**|**3,358**|**92.35%**|

When the 393 Uncertain FOLIO examples are counted as False, the FOLIO accuracy is 82.48% over 1,204 examples. Under this setting, the combined accuracy is 92.53% over 4,029 examples.

### Reasoning Time

The report measures solver-call time only; file reading and English-to-logic translation are excluded.

* RuleTaker: 0.000013 ± 0.000003 seconds per story
* CounterBench: 0.0022 ± 0.0048 seconds per story
* FOLIO without Uncertain examples: 0.0236 ± 0.0597 seconds per story
* Full FOLIO evaluation: 0.0297 ± 0.0620 seconds per story

## 5\. Important Implementation Notes

* RuleTaker and CounterBench English inputs are translated according to the patterns used in their datasets.
* FOLIO's annotated logical formulas are used directly rather than translating the natural-language stories again.
* A solver result of unknown is stored as Unresolved. It is not silently converted into False.
* The report evaluates the released dataset labels and discusses cases where the logical procedure and the released label do not agree.

## 6\. Conclusion

The project uses classical logical reasoning procedures to evaluate entailment across three datasets. Forward chaining performs well on the ground Horn-rule structure of RuleTaker, fast resolution is used for CounterBench's counterfactual questions, and Z3 handles the first-order formulas in FOLIO.

Across RuleTaker, CounterBench, and the binary portion of FOLIO, the system achieves 92.35% accuracy over 3,636 examples. With FOLIO's Uncertain examples counted as False, the combined accuracy is 92.53% over 4,029 examples. Results that the solver cannot decide remain marked as Unresolved.

## 7\. References

1. RuleTaker dataset: https://huggingface.co/datasets/jise/ruletaker
2. CounterBench dataset: https://huggingface.co/datasets/CounterBench/CounterBench
3. FOLIO dataset: https://huggingface.co/datasets/yale-nlp/FOLIO




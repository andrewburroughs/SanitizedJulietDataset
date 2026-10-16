# Sanitized Paired Juliet Vulnerability Benchmark

## Overview

This repository contains a sanitized, matched-pair vulnerability-classification benchmark derived from the NIST Juliet Test Suite. The dataset was created to support controlled evaluation of large language models (LLMs) and other automated vulnerability-classification systems while reducing benchmark artifacts that can reveal the expected label.

The combined dataset contains **29,854 function- or method-level samples** arranged into **14,927 vulnerable/non-vulnerable pairs** across **161 CWE categories** and three programming languages:

| Language | Samples |
|---|---:|
| C | 13,152 |
| Java | 7,942 |
| C# | 8,760 |
| **Total** | **29,854** |

The classes are exactly balanced:

| Label | Meaning | Samples |
|---|---|---:|
| `1` | Vulnerable | 14,927 |
| `0` | Non-vulnerable / fixed | 14,927 |

The primary dataset file is:

```text
juliet_all_eval_paired.jsonl
```

Each line is a single JSON object representing one extracted function or method.

## Motivation

Security benchmarks such as Juliet provide known vulnerable and non-vulnerable examples with precise ground truth. However, the original benchmark source can contain information that makes vulnerability classification artificially easy for a language model, including comments, CWE-related identifiers, Juliet-specific naming conventions, and labels such as `good` and `bad`.

This dataset was created to reduce those sources of label leakage and to provide matched vulnerable and non-vulnerable examples from the same Juliet testcase family. The goal is to evaluate whether a model can classify the **code itself**, rather than recognize benchmark metadata.

## Dataset Construction

The benchmark was derived from cleaned Juliet Test Suite source code for C, Java, and C#.

Source files were parsed with Tree-sitter and reduced to function- or method-level units:

- **C:** `function_definition`
- **Java:** methods and constructors
- **C#:** methods, constructors, and local functions

For each eligible Juliet testcase, one vulnerable and one non-vulnerable implementation were selected to form a matched pair.

Pairs were grouped using:

```text
(language, CWE, testcase)
```

When multiple eligible functions existed within a group, the longest candidate was selected. Pair selection was deterministic.

The resulting benchmark contains:

```text
14,927 vulnerable/non-vulnerable pairs
29,854 total samples
161 CWE categories
```

C++ is not included in this release because the Juliet C++ testcase naming structure did not produce valid vulnerable/non-vulnerable pairs under the pairing strategy used for the C, Java, and C# datasets.

## Sanitization

The `code_clean` field is the intended representation for vulnerability-classification experiments.

The cleaning process was designed to remove obvious benchmark-specific label leakage while preserving code structure and security-relevant semantics. It includes:

- removal of line and block comments;
- removal of Juliet control tokens such as `OMITGOOD`, `OMITBAD`, and `INCLUDEMAIN`;
- neutralization of the target function or method name;
- normalization of non-keyword identifiers where appropriate;
- replacement of string contents with neutral placeholders;
- preservation of language keywords, syntax, relevant function calls, and selected type or macro information needed to retain program meaning.

The transformation is intentionally not a complete semantic anonymization step. Some identifiers, types, calls, or macros may be retained when removing them would materially alter the program structure or meaning.

### Important

The dataset also contains a `code_raw` field with the extracted, unsanitized Juliet function or method. **Do not use `code_raw` as model input when reproducing the sanitized benchmark evaluation.** It may contain comments, function names, or other benchmark artifacts that reveal the expected label.

## Matched-Pair Design

Each pair contains:

- one vulnerable sample (`label = 1`);
- one non-vulnerable sample (`label = 0`);
- the same language;
- the same CWE;
- the same Juliet testcase family.

The following fields identify the pair:

```text
pair_id
pair_key_language
pair_key_cwe
pair_key_testcase
pair_role
pair_index_within_cwe
```

`pair_role` is either:

```text
vulnerable
fixed
```

The matched design is intended to reduce confounding differences between independently sampled vulnerable and non-vulnerable examples.

## Record Format

Each JSONL record contains the following fields.

### Identification and Metadata

| Field | Description |
|---|---|
| `id` | Unique sample identifier |
| `language` | Programming language (`c`, `java`, or `csharp`) |
| `cwe` | CWE identifier associated with the Juliet testcase |
| `testcase` | Juliet testcase family |
| `variant` | Original Juliet variant indicator |
| `label` | Binary ground-truth label: `1` vulnerable, `0` non-vulnerable |
| `source_path` | Original source path used during extraction |
| `span` | Byte offsets of the extracted function/method in the source file |

### Code

| Field | Description |
|---|---|
| `code_raw` | Extracted source before sanitization |
| `code_clean` | Sanitized source intended for benchmark evaluation |

### Pair Metadata

| Field | Description |
|---|---|
| `paired_eval` | Indicates that the sample belongs to the paired benchmark |
| `pair_id` | Identifier shared by both members of a pair |
| `pair_key_language` | Language used in the pairing key |
| `pair_key_cwe` | CWE used in the pairing key |
| `pair_key_testcase` | Testcase used in the pairing key |
| `pair_role` | `vulnerable` or `fixed` |
| `pair_selection_strategy` | Candidate-selection method used when constructing the pair |
| `pair_index_within_cwe` | Pair index within the CWE |

## Recommended Evaluation Use

For vulnerability-classification experiments:

1. Use `code_clean` as the model input.
2. Use `label` as the ground truth.
3. Preserve the provided matched pairs.
4. Report results on the complete benchmark rather than resampling individual functions unless a new experimental design explicitly requires it.
5. Report at least accuracy, precision, recall, F1 score, per-language performance, per-CWE performance where support is sufficient, and the invalid/unparseable output rate for generative models.

When testing prompt sensitivity, inference parameters and evaluation samples should be held constant so that observed differences can be attributed to prompt framing.

## Manual Validation

Before publication, users are encouraged to manually inspect a subset of pairs to verify that:

- vulnerable and non-vulnerable labels correspond to the intended Juliet variants;
- comments and obvious label-revealing artifacts have been removed from `code_clean`;
- sanitization has not corrupted code structure;
- paired samples correspond to the same testcase family;
- the vulnerable/non-vulnerable distinction remains visible in the program logic rather than only in metadata.

## Provenance

This dataset is derived from the **NIST Juliet Test Suite**, distributed through the **NIST Software Assurance Reference Dataset (SARD)**.

Relevant references include:

- F. E. Boland Jr. and P. E. Black, “The Juliet 1.1 C/C++ and Java Test Suite,” *Computer*, vol. 45, no. 10, pp. 88–90, 2012.
- P. E. Black, “A Software Assurance Reference Dataset: Thousands of Programs With Known Bugs,” *Journal of Research of the National Institute of Standards and Technology*, vol. 123, 2018.

Please cite the original Juliet/SARD sources when using this derived benchmark.

## Citation

If you use this dataset, please cite the associated paper once publication information is available.

## Licensing and Redistribution

This dataset contains transformed material derived from the NIST Juliet Test Suite. Users and redistributors should review the applicable NIST/SARD and Juliet distribution terms before redistributing derived source-code content.

This README does not itself grant rights beyond those applicable to the original source material and any separately stated license for scripts or metadata included in this repository.
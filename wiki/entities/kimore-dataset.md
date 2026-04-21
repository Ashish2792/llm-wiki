---
title: KIMORE Dataset
type: entity
entity_type: product
tags: [dataset, rehabilitation, skeleton, kinect, exercise-assessment, benchmark, clinical]
last_updated: 2026-04-21
---

# KIMORE Dataset

## Overview

KIMORE (KInematic assessment of MOvement and clinical scores for Remote monitoring of physical REhabilitation) is a publicly available skeleton dataset for rehabilitation exercise quality assessment. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]] It contains approximately 1,200 sequences of 5 rehabilitation exercises collected from 78 subjects using a Microsoft Kinect v2 sensor. {direct} [[sources/capstone-rehab-exercise-assessment#background]] KIMORE is notable for including real patients (not just healthy subjects simulating errors) and providing continuous clinical quality scores in addition to binary labels. {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]

## Relevance to This Wiki

KIMORE is the second major rehabilitation benchmark dataset relevant to the capstone project domain. It was referenced in the literature review comparison of rehabilitation datasets, and its characteristics — larger subject population (78 vs 10), real patients, and clinical scoring — make it complementary to UI-PRMD. It is listed in the capstone's connections as a future source for enriching rehabilitation benchmark comparisons.

## Key Facts

- **Subjects**: ~78 subjects including healthy controls and neurological patients {direct} [[sources/capstone-rehab-exercise-assessment#background]]
- **Exercises**: 5 rehabilitation exercises {direct} [[sources/capstone-rehab-exercise-assessment#background]]
- **Sequences**: ~1,200 total {direct} [[sources/capstone-rehab-exercise-assessment#background]]
- **Sensor**: Microsoft Kinect v2 (3D skeleton coordinates) {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]
- **Labels**: Continuous clinical quality scores (from physiotherapist assessment) as well as binary labels {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]
- **Population**: Includes real patients (stroke, Parkinson's disease, etc.) alongside healthy subjects — unlike UI-PRMD which uses only healthy subjects simulating errors {paraphrase} [[sources/capstone-rehab-exercise-assessment#background]]
- **Availability**: Publicly available

## Appearances

- [[sources/capstone-rehab-exercise-assessment]] — cited in literature review and dataset comparison; proposed for future enrichment of rehabilitation benchmark comparisons

## Connections

- [[entities/ui-prmd-dataset]] — the other major rehabilitation skeleton benchmark; KIMORE is larger in subjects but fewer exercises
- [[entities/microsoft-kinect]] — the sensor used to collect KIMORE
- [[concepts/skeleton-based-action-recognition]] — KIMORE appears in the benchmark datasets comparison table

## Open Questions

- Why wasn't KIMORE used as an additional training source in the capstone? The source mentions it in the literature review but does not detail attempts to use it. {uncertain} [[sources/capstone-rehab-exercise-assessment#background]]
- Whether clinical quality scores (rather than binary labels) would improve model training is unexplored in the capstone. {uncertain} [[sources/capstone-rehab-exercise-assessment#background]]

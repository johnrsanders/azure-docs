---
title: Top-level entities in managed feature store
titleSuffix: Azure Machine Learning
description: Learn about how Azure Machine Learning uses managed feature stores to create data transformation features and make these features available for training and deployment.
author: rsethur
ms.author: seramasu
ms.reviewer: franksolomon
ms.service: machine-learning
ms.subservice: mldata 
ms.topic: conceptual
ms.date: 05/23/2023 
ms.custom: template-concept, build-2023
---

# Understanding top-level entities in managed feature store

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

This document describes the top level entities in the managed feature store. 

:::image type="content" source="media/concept-managed-feature-store/concepts.png" alt-text="Diagram depicting the main components of managed feature store.":::

For more information on the managed feature store, see [What is managed feature store?](concept-what-is-managed-feature-store.md)

## Feature store
You can create and manage feature sets through a feature store. Feature sets are a collection of features. You can optionally associate a materialization store (offline store connection) with a feature store in order to precompute the features regularly and persist it. This can make feature retrieval during training or inference faster and more reliable.

For configuration details, see [CLI (v2) feature store YAML schema](reference-yaml-feature-store.md)

## Entities
Entities encapsulate the index columns for logical entities in an enterprise. Examples of entities are account entity, customer entity, etc. Entities help enforce best practice that same index column definitions are used across feature sets that use the same logical entities.

Entities are typically created once and reused across feature-sets. Entities are versioned.

For configuration details, see [CLI (v2) feature entity YAML schema](reference-yaml-feature-entity.md)

## Feature set specification and asset
Feature sets are a collection of features that generated by applying transformations on source system data. Feature sets encapsulate a source, the transformation function, and the materialization settings. Currently we support PySpark feature transformation code.

You start by creating a feature set specification. A feature set specification is a self-contained definition of feature set that can be developed and tested locally.

A feature set specification typically consists of the following parameters:
1. `source`: What source(s) does this feature map to
1. `transformation` (optional): The transformation logic that needs to be applied to the source data to create features. In our case, we use spark as the supported compute.
1. Names of the columns represent the `index_columns` and the `timestamp_column`: This is required when users try to join feature data with observation data (more about this later)
1. `materialization_settings`(optional): Required if you want to cache the feature values in a materialization store for efficient retrieval.

Once you have developed and tested the feature set spec in your local/dev environment, you can register it as a feature set asset with the feature store in order to get managed capabilities like versioning and materialization.

For details on the feature set YAML specification, see [CLI (v2) feature set specification YAML schema](reference-yaml-featureset-spec.md)

## Feature retrieval specification
A feature retrieval specification is a portable definition of a list of features associated with a model. This can help streamline machine learning model development and operationalizing. A feature retrieval specification is typically an input to the training pipeline (used to generate the training data), which can be packaged along with the model, and will be used during inference to look up the features. It's a glue that integrates all phases of the machine learning lifecycle. Changes to your training and inference pipeline can be minimized as you experiment and deploy.

Using a feature retrieval specification and the built-in feature retrieval component are optional: you can directly use `get_offline_features()` API if you prefer.

For details on the feature retrieval YAML specification, see [CLI (v2) feature retrieval specification YAML schema](reference-yaml-feature-retrieval-spec.md).

## Next steps

- [What is managed feature store?](concept-what-is-managed-feature-store.md)
- [Manage access control for managed feature store](how-to-setup-access-control-feature-store.md)

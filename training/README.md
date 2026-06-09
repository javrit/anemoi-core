# anemoi-training

[![Documentation Status](https://readthedocs.org/projects/anemoi-training/badge/?version=latest)](https://anemoi-training.readthedocs.io/en/latest/?badge=latest)


**DISCLAIMER**
This project is **BETA** and will be **Experimental** for the foreseeable future.
Interfaces and functionality are likely to change, and the project itself may be scrapped.
**DO NOT** use this software in any project/software that is operational.

Miscellanous tools for training data-driven weather forecasts.

## Documentation

The documentation can be found at https://anemoi-training.readthedocs.io/.

## Install

Install via `pip` with:

```
$ pip install anemoi-training
```

## Training unconditional diffusion model


This document explains how to set up and run the unconditional diffusion training with Anemoi.


### Key configuration points

#### 1. Creating the graph

It is advised to create the graph before launching training (to avoid re-creating it each time).
The config used is available at :
```
./graphs/graph_multiscale.yaml
```

It is necessary to replace the path to the dataset :

```yaml
nodes:
  # Data nodes
  data:
    node_builder:
      _target_: anemoi.graphs.nodes.AnemoiDatasetNodes
      dataset:
        - dataset: path/to/dataset # <-- replace here
    attributes:
      area_weight:
        _target_: anemoi.graphs.nodes.attributes.PlanarAreaWeights
        norm: unit-max # options: l1, l2, unit-max, unit-sum, unit-std
```


#### 2. Model — use the unconditional target


In your config, make sure the model target points to the **unconditional** class:


```yaml
model:
  model:
    _target_: anemoi.models.models.AnemoiDiffusionModelEncProcDecUnconditional
    diffusion:
      sigma_max: 1000 # <-- to be defined
  condition: "mean"  # replacing the condition by a constant (mean computed on each point for each variable over the whole training dataset 
```


> ⚠️ Do **not** use the standard `AnemoiDiffusionModelEncProcDec` — it expects conditioning inputs that are not provided in the unconditional setup.


#### 3. Training task — use the unconditional forecaster


```yaml
training:
  model_task: anemoi.training.train.tasks.GraphUnconditionalDiffusionForecaster
  training_approach: probabilistic_high_noise
  multistep_input: 1
```


> ⚠️ Do **not** use `GraphDiffusionForecaster` — it is the conditional variant.


---



### Paths to fill in


The following paths must be updated to match your environment before launching training.


#### Dataset path


```yaml
dataloader:
  dataset:
    - dataset: /path/to/your/dataset.zarr   # <-- update this
```


#### Hardware paths


```yaml
hardware:
  paths:
    data: /path/to/your/dataset/folder      # <-- update this (folder containing the zarr)
    output: /path/to/output-dir             # <-- update this (where checkpoints are saved)
  files:
    dataset: your-dataset-name.zarr         # <-- update this (filename only, no path)
    graph: /path/to/graph.pt                # <-- update this
```


---


A full minimal config example is available at path : ./training/src/anemoi/training/config/training_example.yaml


---

## Finetuning unconditional diffusion model

The difference between training and finetuning is the noise scheduler.

```yaml
training:
  # Set max_epochs or max_steps. Training stops at the first limit reached.
  max_steps: 150000
  model_task: anemoi.training.train.tasks.GraphUnconditionalDiffusionForecaster
  training_approach: probabilistic_low_noise # <-- change here 
  multistep_input: 1
  fork_run_id: fork_run_id
  load_weights_only: True
```

A full minimal config example is available at path : ./training/src/anemoi/training/config/finetuning_example.yaml


## License

```
Copyright 2024, Anemoi contributors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

In applying this licence, ECMWF does not waive the privileges and immunities
granted to it by virtue of its status as an intergovernmental organisation
nor does it submit to any jurisdiction.
```

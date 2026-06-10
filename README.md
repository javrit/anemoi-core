# anemoi-core

[![Static Badge](https://github.com/ecmwf/codex/raw/refs/heads/main/Project%20Maturity/incubating_badge.svg)](https://github.com/ecmwf/codex/raw/refs/heads/main/Project%20Maturity#incubating)


> \[!IMPORTANT\]
> This software is **Incubating** and subject to ECMWF's guidelines on [Software Maturity](https://github.com/ecmwf/codex/raw/refs/heads/main/Project%20Maturity).

A mono-repo containing core training and modelling functionality for Anemoi, providing the packages `anemoi-training`, `anemoi-models`, and `anemoi-graphs`.

Anemoi training contains miscellanous tools for training data-driven weather forecasts. Anemoi models contains the core model components that build the architecture of each data-driven NWP model. Anemoi graphs provides tools to build graphs for data-driven forecasts.

## Documentation

The documentation can be found at:

- https://anemoi-training.readthedocs.io/
- https://anemoi-models.readthedocs.io/
- https://anemoi-graphs.readthedocs.io/


## Install

Install via `pip` with:

```
$ pip install anemoi-training
$ pip install anemoi-models
$ pip install anemoi-graphs
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
      sigma_max: 1000
      log_normal_mean: -1.2
      log_normal_std: 1.2
  processor:
    num_layers: 16
  condition: "mean"  # options: "zero", "mean" ; set to zero if you have not computed the mean, and mean and std as explained in the README
  condition_files: #files used when using the condition = "mean", 
    mean_point: ??? # mean of each variable on each point (shape = (1 , 1, lat * lon, variables)) ; npy file
    mean_vars: ??? # spatial mean of each variable : shape = (78,) ; npy file
    std_vars: ??? # spatial std of each variable : shape = (78,) ; npy file 
```

> ⚠️ Do **not** use the standard `AnemoiDiffusionModelEncProcDec` — it expects conditioning inputs that are not provided in the unconditional setup.


Two options are available for the condition : mean and zero. 

When using the mean condition, the condition files can be set to zero and the condition (in AnemoiDiffusionModelEncProcDec) is replaced by the normalized mean tensor of each variable. Thus, it is necessary to compute the mean of each variable on each point (file = mean_point), under a npy file. The mean_vars and mean_std files are the mean and std of each variable, computed spatially, and can be found within the zarr dataset.  

When using the zero condition, the condition files can be set to zero and the condition (in AnemoiDiffusionModelEncProcDec) is replaced by a null tensor.

#### 3. Training task — use the unconditional forecaster


```yaml
training:
  model_task: anemoi.training.train.tasks.GraphUnconditionalDiffusionForecaster
  training_approach: probabilistic_high_noise
  multistep_input: 1
```


> ⚠️ Do **not** use `GraphDiffusionForecaster` — it is the conditional variant.


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

---



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

## Training unconditional diffusion model


This document explains how to set up and run the unconditional diffusion training with Anemoi.

First, you need to clone the environment used (github: https://github.com/javrit/anemoi-env), and place yourself in the folder to launch training/finetuning.

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
    _target_: anemoi.models.models.AnemoiDiffusionModelEncProcDecUnconditional # <-- Unconditional model
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

The configuration used for our experiments is available at path : ./training/src/anemoi/training/config/training_config_used.yaml
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

The configuration used for our finetuning is available at path : ./training/src/anemoi/training/config/finetuning_config_used.yaml

---

# spell-test

This repo tests out some spell.run features.

## General organization

* The CLI is organized with UNIX shell -like commands with the spell prefix. E.g. `spell ls`, `spell cp`.
* Runs are given an incremental ID (the model run number) which counts up from 1.
* Run "resources" are inputs, and run "outputs" are outputs.
* Run-specific object storage is part of a hierarchy of folders which also includes public assets (which probably shadow out to a different bucket, or something) and assets that you have uploaded by hand.
* The final argument to `spell run` is the instruction to be run, in quotes (e.g. `"python train.py"`).

## Run environment definition

* The default environment has a specific set of frameworks installed. Specifying the `--framework` option when executing a run narrows the pick down to a specific framework. I don't immediately see an option for setting a specific framework version.
* Alternatively, one can specify a `--conda-file` path to a `conda` environment specification.
* `apt` and `pip` are available as CLI arguments. There is no `conda` CLI argument equivalent, but see the above bullet point about providing a `conda`-based environment definition.
* Power customization is via the `--from` flag, which takes a public DockerHub image as input.
* Further pre-training or pre-serving customizations to the environment can also be provided as runtime instructions.
* Runs build Docker images, and those images are cached.
* The `-m` (mount) flag allows for mounting file resources from previous runs to the current run. E.g. `-m runs/42:/mnt/model` to boot a previous model into the current. So to chain a model serving runs over model building runs, the pattern is: (1) build a model in a specific run, saving the model artifact as an output (2) .

## Workspaces

* Workspaces are interfaces to remote Jupyter Notebook or to JupyterLab instances. This is the Spell equivalent to the Jupyter component of AWS SageMaker, pretty much.
* Under the hood, workspaces are just long-held runs; you reuse all of the same infrastructure. However, workspace executions are not listed as runs in the run interface.
* One big difference from a user perspective is that the default mode for initializing a workspace is to use the web GUI; the same is not true of runs, which are mostly initialized from the CLI.

## Resources

* Resources are the generic name for datasets, models, or any other files that can be made available to a run.
* Resources in the free tier are uploaded to their servers. Resources in the enterprise tiers are in S3 object storage (or GCP?). Resources are persistant, mountable, and downloadable (as described in the previous sections).
* Run-generated resources are named to a path including the generative model run number. Commonly used or just-in-time linked resources can be managed using symlinks, which path a well-known alias to an absolute path to a resource.

## Integrations

* The free tier is basically a play space. They want you using (and paying for) your own AWS or GCP infra that Spell is running alongside.
* Spell itself uses Kubernetes for its infrastructure, e.g. EKS or GKE, because duh.
* There are aliases for instance types, e.g. `cpu`, `cpu-big`, `cpu-huge`.
* Automatic resizing is provided automatically, with user-configured toggles on how much the cluster can flex. This is a mask over Kubernetes resizing facilities.
* Spell keeps a fleet of machines ready to avoid cold start. It does some form of load prediction to achieve this.
* Cluster updates are integrated into the CLI via `spell cluster update`.
* A GH hook is provided which allows for running from private repositories associated with the hook via `--github-url`.

## Hyperparameter search

* It exists. Parameterization is handled via CLI input, e.g. `spell hyper grid --param rate=0.001,0.01,0.1,1 --param layers=2,3,5,10`.
* I still see this is as a sales-oriented feature. Automated hyperparameter search is something that everyone wants to be able to say that they have, and that's fine. But beyond the brute-force hyperparameter search use case, I still think that hyperparemeter search is best executed in code.
  
  Why? The reduced overhead of a platform like Spell is attractive for setting up the compute job that does model computation because the compute used to generate a model is an implementation detail, from the perspective of the end user. Hyperparameter tuning is *not* an implementation detail; it is a fundamental part of the procedure of training a model in the first place. As such, it is something that asks for maximum control and visibility and minimum magic, from a user perspective.

## Model serving

* TensorFlow Serving support is integrated in, and they have a custom function that maybe they themselves wrote that does roughly the same thing.
* This is run via `spell model-servers create` or similar.

## Distributed training

* Distributed training is available from the CLI.
* Distributed training requires code changes; it is not "free". Spell has evidently already set up their compute with `horovod` support baked in. [See an example of how this is done here](https://github.com/horovod/horovod/blob/master/examples/keras_mnist.py). Overall this doesn't seem to require *too* many modifications to the model code, and it supports all of the big frameworks, so I'm a fan.

## TensorBoard

* Inline support for TensorBoard exists. Launchable from a window on the run console.

## Metrics

* Hardware and model metrics are tracked in the web interface.
* You can also custom-define your own metrics, which will similarly register to the web interface.

## Workflows

* Workflows are runs the create other runs. They're more complicated and I just glanced over them.
* Basically, runs (what I call experiments in my own writing) are the atomic unit of value in Spell; good.

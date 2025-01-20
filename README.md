# Snakemake profiles for Slurm schedulers

> [!note]
> This repository was forked to account for the change of [execution backends into plugins](https://snakemake.readthedocs.io/en/stable/getting_started/migration.html#command-line-interface) in Snakemake v8 that made the previous profile unusable.

This repository offers two profiles to run Snakemake with Slurm schedulers:
1. A `slurm-generic` profile: this is a generic profile that was adapted to Slurm. It can be heavily customised and **this is the profile we recommend**. It is strongly influenced by the Slurm profile in Snakemake's official [profiles repository](https://github.com/Snakemake-Profiles/slurm), which is great, but is missing some features and documentation. This profile was developed to address these points and implement the following features:
    - No cookiecutter dependency
    - Comprehensive documentation making the profile easy to use
    - "Smart" handling of partitions
    - Customability through YAML files
    - "Clean" implementation to allow adding new features efficiently
2. A `slurm` profile: it was specifically developed for Slurm and Snakemake v8, but is still missing some functionalities and features that are available in the generic profile. It is slightly easier to use but offer less options (or make them less easily accessible). The full plugin documentation can be found [here](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html) but unfortunately, it contains several conflicting pieces of information on its usage.

> [!important]
> This is a work in progress. Feature requests, issues, and pull requests welcome!
<br>

## Table of contents

<!-- MarkdownTOC -->

- [Installation](#installation)
    + [Requirements](#requirements)
        * [Note for UNIL/CHUV HPC users](#note-for-unilchuv-hpc-users)
    + [Installing the profiles](#installing-the-profiles)
    + [Updating the profiles](#updating-the-profiles)
    + [Resetting the profiles](#resetting-the-profiles)
- [Usage](#usage)
    + [Using a profile](#using-a-profile)
    + [Specifying parameter values](#specifying-parameter-values)
        * [Global Slurm parameters](#global-slurm-parameters)
            - [`slurm-generic` profile](#slurm-generic-profile)
            - [`slurm` profile](#slurm-profile)
        * [Rule-specific resources](#rule-specific-resources)
        * [Rule-specific CPUs/threads](#rule-specific-cpusthreads)
        * [Implementing custom slurm parameters \(`slurm-generic` only\)](#implementing-custom-slurm-parameters-slurm-generic-only)
        * [Other Snakemake parameters](#other-snakemake-parameters)
    + [Blacklisting partitions \(`slurm-generic` only\)](#blacklisting-partitions-slurm-generic-only)
        * [Note for UNIL/CHUV HPC users](#note-for-unilchuv-hpc-users-1)
    + [Adjusting update rate for partition information \(`slurm-generic` only\)](#adjusting-update-rate-for-partition-information-slurm-generic-only)
- [Troubleshooting](#troubleshooting)
    + [Job stuck in PD state](#job-stuck-in-pd-state)

<!-- /MarkdownTOC -->
<br>

## Installation

### Requirements

To function properly, the profiles require the following Python modules:

- `pyyaml`
- `snakemake`
- `snakemake-executor-plugin-cluster-generic` (for the `slurm-generic` profile)
- `snakemake-executor-plugin-slurm` (for the `slurm` profile)

If you are using a dedicated Conda environment for Snakemake (recommended by the [Snakemake developpers](https://snakemake.readthedocs.io/en/stable/getting_started/installation.html#installation-via-conda-mamba)) or a PyPi install of Snakemake, you should already have the first two modules, but otherwise, all the modules are available in Conda and PyPi and can be installed with either:

* PyPi (in a user environment, *e.g.* on a cluster):

    ```bash
    pip install --user snakemake
    pip install --user pyyaml
    pip install --user snakemake-executor-plugin-cluster-generic
    pip install --user snakemake-executor-plugin-slurm
    ```

* Conda/mamba (in a dedicated Snakemake environment):

    ```bash
    mamba create snakemake  # Create a dedicated environment for Snakemake
    mamba activate snakemake  # Activate the environment
    mamba install -c conda-forge -c bioconda snakemake
    mamba install -c conda-forge -c bioconda pyyaml
    mamba install -c conda-forge -c bioconda snakemake-executor-plugin-cluster-generic
    mamba install -c conda-forge -c bioconda snakemake-executor-plugin-slurm
    ```

#### Note for UNIL/CHUV HPC users

By default, Conda will be installed in your home (`/users/<username>`) which has low storage and file number quotas, so we recommend to change the install path to `/work` and use it to create an environment dedicated to Snakemake.

### Installing the profiles

To install the profiles, clone the `snakemake-slurm` repository and run the `INSTALL` script:

```bash
git clone https://github.com/bdsc-tds/snakemake-slurm.git
cd snakemake-slurm
./INSTALL
```

By default, the `slurm` and `slurm-generic` profiles will be installed in `~/.config/snakemake/slurm` and `~/.config/snakemake/slurm-generic`, respectively.

> [!tip]
> You can specify a different name for the profiles. In this case, the profiles will be installed in `~/.config/snakemake/<profile_name>` and `~/.config/snakemake/<profile_name>-generic`:
>
> ```bash
> git clone https://github.com/bdsc-tds/snakemake-slurm.git
> cd snakemake-slurm
> ./INSTALL <profile_name>
> ```

### Updating the profiles

To update the profiles' code while retaining the configuration files (.yaml), pull from the `snakemake-slurm` repository and run the `UPDATE` script:

```bash
# From within the snakemake-slurm directory (not the directory where the profiles were installed)
git pull
./UPDATE
```

> [!tip]
> If you used a name other than the default `slurm` for the profiles, specify the name when you run `UPDATE`:
>
> ```bash
> # From within the snakemake-slurm directory (not the directory where the profiles were installed)
> git pull
> ./UPDATE <profile_name>
> ```

### Resetting the profiles

Updating the profiles will update their code but not their configuration files, so that your configuration is retained. To update all the files (including the configuration files) and reset the partitions info file, pull from the `snakemake-slurm` repository and run the `FULL_UPDATE` script:

```bash
# From within the snakemake-slurm directory (not the directory where the profiles were installed)
git pull
./FULL_UPDATE
```

> [!tip]
> If you used a name other than the default `slurm` for the profiles, specify the name when you run `FULL_UPDATE`:
>
> ```bash
> # From within the snakemake-slurm directory (not the directory where the profiles were installed)
> git pull
> ./FULL_UPDATE (<profile_name>)
> ```

## Usage

### Using a profile

To run Snakemake with a profile, use the runtime parameter `--profile`: `snakemake --profile slurm-generic` or `snakemake --profile slurm` depending on the profile you want to use. Do not forget to replace `slurm` by `<profile_name>` if you installed the profile under a different name. For more information on Snakemake profiles, check out the [official Snakemake documentation](https://snakemake.readthedocs.io/en/stable/executing/cli.html?#profiles).

### Specifying parameter values

#### Global Slurm parameters

##### `slurm-generic` profile

You can specify a value for any `sbatch` parameter by adding it to the `global_options` field of `slurm.yaml` in `~/.config/snakemake/<profile_name>-generic` (see [here](https://slurm.schedmd.com/sbatch.html) for a list of available `sbatch` parameters). The specified value will be used for **all** jobs submitted with this profile, so this should be used to specify very general parameters, like a Slurm account to use for job submission or an email address to receive notifications.

Below is an example of configuration so that all jobs are via the `user_account` account on the `cpu` partition and an email is sent after each job ends:

```yaml
global_options:
    partition: 'cpu'
    account: 'user_account'
    mail-type: 'END'
    mail-user: 'user@domain.com'
```

> [!warning]
> Although this is not recommended, you can also use the `global_options` field to specify the same resource usage for all the jobs generated by your workflow (using the `sbatch` resource parameters and syntax). Here is an example of configuration to request 2 CPUs, 1GB, and 8h runtime (D-H:M:S format, see more [here](https://slurm.schedmd.com/sbatch.html#OPT_time) about time format) for every job:
>
> ```yaml
> global_options:
>     cpus-per-task: 2
>     mem: 1G
>     time: 0-8:00:00
> ```
> However, keep in mind that it is more efficient (and easier!) to specify the resource usage for each rule individually, either using a more fine-tuned configuration (see [below](#rule-specific-resources)) or [inside the rules](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#threads) themselves.

##### `slurm` profile

The same thing can be done with the `slurm` profile: to specify a value for a `sbatch` parameter, add it to the `default-resources` field of `config.yaml` in `~/.config/snakemake/<profile_name>` (see [here](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html#advanced-resource-specifications) for a list of the `sbatch` parameters available in this type of profile). Note that the parameter names are slightly different from the `sbatch` ones in the `slurm-generic` profile because they are actually coming from Snakemake, not from `sbatch`.

Below is an example of configuration so that all jobs are requesting 2 CPUs, 1G of memory, 480 min (= 8h) runtime and submitted via the `user_account` account on the `cpu` partition:

```yaml
default-resources:
    slurm_partition: 'cpu'
    slurm_account: 'user_account'
    cpus_per_task: 2
    mem_mb: 1024
    runtime: 480
```

A lot of the `sbatch` parameters are not appearing in the [previous table](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html#advanced-resource-specifications). They can still be specified with the `slurm_extra` parameter, but they need to be written as a string enclosed by both single quotes `'` **and** double quotes `"`, which is much less convenient to read and write compared to the `global_options` field in the `slurm-generic` profile.

With the previous example about email notifications, the configuration would be:

```yaml
# Configuration to send an email after each job ends
default-resources:
    slurm_extra: "'--mail-type=END --mail-user=user@domain.com'"
```

#### Rule-specific resources

While it is standard to define rule resources directly in the [workflow code](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#resources), both profiles offer the option to specify them with the `set-resources` field in the `config.yaml` file in `~/.config/snakemake/<profile_name>` and `~/.config/snakemake/<profile_name>-generic`.

Below is an example of a rule called `test` that is submitted to a partition different from the default (`other_partition`), and requests 1024 MB of memory and 60 minutes of runtime via the `slurm-generic` profile:

```yaml
set-resources:
    test:
        partition: 'other_partition'
        mem_mb: 1024
        runtime: 60
```

And here is the same case with the `slurm` profile:

```yaml
set-resources:
    test:
        slurm_partition: 'other_partition'
        slurm_extra: "'--mail-type=BEGIN --mail-user=user@domain.com'"
        mem_mb: 1024
        runtime: 60
```

Note the use of `slurm_partition` instead of `partition`: this is because the `slurm` profile relies on the [executor plugin keywords](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html#advanced-resource-specifications) while the `slurm-generic` profile relies on the [Snakemake keywords](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#resources). Also note the presence of the `slurm_extra` parameter (here, to send an email when the rule starts), that is not available in the `slurm-generic` profile.

> [!important]
> It is advised to use the `resources` keyword directly in a Snakemake rule to specify runtime in minutes and memory in MB using `mem_mb` and `runtime`, as they are the standard keywords and units expected by Snakemake. They also allow Snakemake to resubmit the job with higher requirements in case of failure (although it needs to be implemented in the workflow using the `attempt` variable). Such a rule would look like this:
>
> ```python
> rule example:
>     output: 'example.txt'
>     resources:
>         mem_mb = 1024,
>         runtime = 60
>         # runtime = lambda wildcards, attempt: 60 * attempt
>     shell:
>         'echo "example" > {output}'
> ```
> In this example, both profiles will check Snakemake's jobscript to find the required resources and automatically submit the job with what was requested.

#### Rule-specific CPUs/threads

Setting the number of CPUs/threads per rule can be done in the `set-resources` field, but it is more common to use the `set-threads` field. Both profiles can do this in the `config.yaml` file in `~/.config/snakemake/<profile_name>` and `~/.config/snakemake/<profile_name>-generic`.

Below is an example of a rule called `test` that requests 24 CPUs to run:

```yaml
set-threads:
    test:
        threads: 24
```

> [!important]
> Similarly to memory and runtime, it is advised to use the `threads` keyword directly in a Snakemake rule to specify the number of threads. It also allows Snakemake to resubmit the job with higher requirements in case of failure (although it needs to be implemented in the workflow using the `attempt` variable). Such a rule would look like this:
>
> ```python
> rule example:
>     output: 'example.txt'
>     threads: 24
>     # threads: lambda wildcards, attempt: 24 * attempt
>     shell:
>         'echo "example" > {output}'
> ```
> In this example, both profiles will check Snakemake's jobscript to find the required number of CPUs and automatically submit the job with what was requested.

#### Implementing custom slurm parameters (`slurm-generic` only)

You can implement additional parameters by adding an entry to the `options` field in the file `slurm.yaml` in `~/.config/snakemake/<profile_name>-generic`, with the format:

```yaml
options:
    <option_name>: <slurm flag>={}
```

In this case, `<option_name>` should match the name of the Snakemake rule option, and `<slurm_flag>` is the flag used to specify this option with the `sbatch` command. The `{}` will be substituted for the option's value if the value was specified in the Snakemake rule.

Below is an example implementing the `--tmp` Slurm flag (minimum amount of temporary disk space) using a parameter `tmp_disk` in Snakemake:

**slurm.yaml:**

```yaml
options:
    tmp_disk: --tmp={}
```

**Snakefile:**

```python
rule example:
    output: 'example.txt'
    params:
        tmp_disk: 4G
    shell:
    'echo "example" > {output}'
```

Here, when Snakemake submits the jobs, it will request 4GB of temporary disk space only for the jobs generated by the `example` rule.

In the Snakemake rule, option values can be specified:
- Within the `resources` keyword
- Within the `params` keyword
- At the rule level (*e.g.* `threads`)

The profile will first look for the options in `resources`, then `params`, then at the rule level.

#### Other Snakemake parameters

The profiles can also be used to set a default for each option of the Snakemake command line interface (see [here](https://snakemake.readthedocs.io/en/stable/executing/cli.html#all-options) for a list of available Snakemake parameters). For this, option `--some-option` becomes `some-option:` in the `config.yaml` file in `~/.config/snakemake/<profile_name>` and `~/.config/snakemake/<profile_name>-generic`.

For example, to restart failing jobs once (`--restart-times` option in Snakemake), you can add this line to `config.yaml`:

```yaml
restart-times: 1
```

In both profiles, we provided reasonable default settings for a few options. Feel free to edit them and add more if needed:
- `jobs`
- `local-cores`
- `max-jobs-per-timespan`
- `max-status-checks-per-second`
- `restart-times`

### Blacklisting partitions (`slurm-generic` only)

If you don't provide one, the `slurm-generic` profile automatically detects all partitions available on the cluster and selects the one with highest priority that satisfies the resources requirements. If you do not want Snakemake to submit to specific partitions at all, you can specify these in the `blacklist` field in the file `slurm.yaml` in `~/.config/snakemake/<profile_name>-generic`.

For example, to avoid submitting jobs to a partition called `interactive`:

**Example**:
```yaml
blacklist:
    - interactive
```

#### Note for UNIL/CHUV HPC users

Since this profile is mainly used by users of the UNIL/CHUV HPC, we share here our configuration for this platform:

```yaml
# Partitions to remove from submit list
blacklist:
  - interactive
  - gpu
```

If you use a GPU for some steps in your workflows, do not blacklist `gpu`.

### Adjusting update rate for partition information (`slurm-generic` only)

By default, the `slurm-generic` profile will scan for partitions the first time it is ever used by Snakemake and will store data about available partitions in a file `partitions.yaml` in `~/.config/snakemake/<profile_name>-generic`. It will then check again and update the information every *N* days (1 day by default). Note that the information is updated only when the profile is used by Snakemake. You can change the partitions data file name as well as the update rate in the `scheduler` field in the file `slurm.yaml`.

## Troubleshooting

### Job stuck in PD state

You can get information on why a job is stuck in the 'NODELIST(REASON)' field in the output of `squeue -u <username>`.

In some cases, partition permissions are not configured properly and `slurm-generic` will try to submit jobs to a partition that is not available for you. There is no official reason for this, but it may contain the word 'Permission' somewhere. In this case, check which partition the job was submitted to with `scontrol show job <job_id>` and add this partition to the [blacklist](#blacklisting-partitions-slurm-generic-only).

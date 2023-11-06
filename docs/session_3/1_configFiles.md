# Configuration basics

## Configuring nf-core pipelines

!!! clipboard-list "Objectives"

    - Learn how to customize the execution of an nf-core pipeline.
    - Customize a toy example of an nf-core pipeline.



## Configuration

Each nf-core pipeline comes with a set of “sensible defaults”. While the defaults are a great place to start, you will almost certainly want to modify these to fit your own purposes and system requirements.

**You do not need to edit the pipeline code to configure nf-core pipelines.**

When a pipeline is launched, Nextflow will look for configuration files in several locations. As each source can contain conflicting settings, the sources are ranked to decide which settings to apply. Configuration sources are reported below and listed in order of priority:

1. Parameters specified on the command line (`--parameter`)
2. Parameters that are provided using the `-params-file` option
3. Config file that are provided using the `-c` option
4. The config file named `nextflow.config` in the current directory
5. The config file named `nextflow.config` in the pipeline project directory
6. The config file `$HOME/.nextflow/config`
7. Values defined within the pipeline script itself (e.g., `main.nf`)

!!! warning

    nf-core pipeline parameters must be passed via the command line (`--<parameter>`) or Nextflow `-params-file` option. Custom config files, including those provided by the `-c` option, can be used to provide any configuration **except** for parameters.

Notably, while some of these files are already included in the nf-core pipeline repository (e.g., the `nextflow.config` file in the nf-core pipeline repository), some are automatically identified on your local system (e.g., the `nextflow.config` in the launch directory), and others are only included if they are specified using `run` options (e.g., `-params-file`, and `-c`).

Understanding how and when these files are interpreted by Nextflow is critical for the accurate configuration of a pipelines execution.

## Parameters

**Parameters** are pipeline specific settings that can be used to customise the execution of a pipeline. 

Every nf-core pipeline has a [full list of parameters](https://nf-co.re/sarek/3.2.3/parameters) on the nf-core website. When viewing these parameters online, you will also be shown a description and the type of the parameter. Some parameters will have additional text to help you understand when and how a parameter should be used.

Parameters and their descriptions can also be viewed in the command line using the `run` command with the `--help` parameter:

```bash
nextflow run nf-core/<workflow> --help
```

!!! question "Exercise" 

    View the parameters for the `christopher-hakkaart/nf-core-demo` pipeline using the command line:

    ??? success "Solution"

        The `christopher-hakkaart/nf-core-demo` pipeline parameters can be printed using the `run` command and the `--help` option:

        ```bash
        nextflow run christopher-hakkaart/nf-core-demo -r main --help
        ```

### Parameters in the command line

At the highest level, parameters can be customised using the command line. Any parameter can be configured on the command line by prefixing the parameter name with a double dash (`--`):

```bash
nextflow nf-core/<workflow> --<parameter>
```

!!! tip "When to use `--` and `-`"

    Nextflow options are prefixed with a single dash (`-`) and pipeline parameters are prefixed with a double dash (`--`).

Depending on the parameter type, you may be required to add additional information after your parameter flag. For example, for a string parameter, you would add the string after the parameter flag:

```bash
nextflow nf-core/<workflow> --<parameter> string
```

!!! question "Exercise" 

    Give the MultiQC report for the `christopher-hakkaart/nf-core-demo` pipeline the name of your **favorite animal** using the [`multiqc_title`](https://github.com/christopher-hakkaart/nf-core-demo/blob/master/nextflow.config#L27) parameter using a command line flag:

    ??? success "Solution"

        Add the `--multiqc_title` flag to your command and execute it. Use the `-resume` option to save time:

        ```bash
        nextflow run christopher-hakkaart/nf-core-demo -profile test,singularity -r main --multiqc_title kiwi -resume
        ```

        In this example, you can check your parameter has been applied by listing the files created in the results folder (`results`):

        ```bash
        ls results/multiqc/
        ```

        `--multiqc_title` is a parameter that directly impacts a result file. For parameters that are not as obvious, you may need to check your `log` to ensure your changes have been applied. You **should not** rely on the changes to parameters printed to the command line when you execute your run:

        ```bash
        nextflow log
        nextflow log <run name> -f "process,script"
        ```

### Default configuration files

All parameters will have a default setting that is defined using the `nextflow.config` file in the pipeline project directory. By default, most parameters are set to `null` or `false` and are only activated by a profile or configuration file.

There are also several `includeConfig` statements in the `nextflow.config` file that are used to include additional `.config` files from the `conf/` folder. Each additional `.config` file contains categorised configuration information for your pipeline execution, some of which can be optionally included:

- `base.config`
    - Included by the pipeline by default.
    - Generous resource allocations using labels.
    - Does not specify any method for software management and expects software to be available (or specified elsewhere).
- `igenomes.config`
    - Included by the pipeline by default.
    - Default configuration to access reference files stored on [AWS iGenomes](https://ewels.github.io/AWS-iGenomes/).
- `modules.config`
    - Included by the pipeline by default.
    - Module-specific configuration options (both mandatory and optional).
- `test.config`
    - Only included if specified as a profile.
    - A configuration profile to test the pipeline with a small test dataset.
- `test_full.config`
    - Only included if specified as a profile.
    - A configuration profile to test the pipeline with a full-size test dataset.

Notably, configuration files can also contain the definition of one or more profiles. A profile is a set of configuration attributes that can be activated when launching a pipeline by using the `-profile` command option:

```bash
nextflow run nf-core/<workflow> -profile <profile>
```

Profiles used by nf-core pipelines include:

- **Software management profiles**
    - Profiles for the management of software using software management tools, e.g., `docker`, `singularity`, and `conda`.
- **Test profiles**
    - Profiles to execute the pipeline with a standardised set of test data and parameters, e.g., `test` and `test_full`.

Multiple profiles can be specified in a comma-separated (`,`) list when you execute your command. The order of profiles is important as they will be read from left to right:

```bash
nextflow run nf-core/<workflow> -profile test,singularity
```

nf-core pipelines are required to define **software containers** and **conda environments** that can be activated using profiles. Although it is possible to run the pipelines with software installed by other methods (e.g., environment modules or manual installation), using Docker or Singularity is more convenient and more reproducible.

!!! tip 

    If you're computer has internet access and one of Conda, Singularity, or Docker installed, you should be able to run any nf-core pipeline with the `test` profile and the respective software management profile 'out of the box'.
    
    The `test` data profile will pull small test files directly from the `nf-core/test-data` GitHub repository and run it on your local system. The `test` profile is an important control to check the pipeline is working as expected and is a great way to trial a pipeline. Some pipelines have multiple test `profiles` for you to try.


### Custom configuration files

Nextflow will also look for files that are external to the pipeline project directory. These files include:

- The config file `$HOME/.nextflow/config`
- A config file named `nextflow.config` in your current directory
- Custom files specified using the command line
    - A parameter file that is provided using the `-params-file` option
    - A config file that are provided using the `-c` option

You don't need to use all of these files to execute your pipeline.

** Parameter files

Parameter files are `.json` files that can contain an unlimited number of parameters:

```json title="my-params.json"
{
   "<parameter1_name>": 1,
   "<parameter2_name>": "<string>",
   "<parameter3_name>": true
}
```

You can override default parameters by creating a custom `.json` file and passing it as a command-line argument using the `-param-file` option.

```bash
nextflow run nf-core/<workflow> -profile test,singularity -r main -param-file <path/to/params.json>
```

## Customizing parameters

Let's take the skills from the previous section and apply them to customise the execution of the Sarek pipeline.

Remember that previoiusly we supplied a series of Sarek pipeline parameters as flags in your run command (`--`). Here, we will package these into a `.json` file and use the `-params-file` option.

!!! question "Exercise"

    Package the parameters from the previous lesson into a `.json` file and run the pipeline using the `-params-file` option:

    ```console
    nextflow run nf-core/sarek \
        --input samplesheet.csv \
        --igenomes_ignore \
        --dbsnp "https://raw.githubusercontent.com/nf-core/test-datasets/modules/data/genomics/homo_sapiens/genome/vcf/dbsnp_146.hg38.vcf.gz" \
        --fasta "https://raw.githubusercontent.com/nf-core/test-datasets/modules/data/genomics/homo_sapiens/genome/genome.fasta" \
        --germline_resource "https://raw.githubusercontent.com/nf-core/test-datasets/modules/data/genomics/homo_sapiens/genome/vcf/gnomAD.r2.1.1.vcf.gz" \
        --intervals "https://raw.githubusercontent.com/nf-core/test-datasets/modules/data/genomics/homo_sapiens/genome/genome.interval_list" \
        --known_indels "https://raw.githubusercontent.com/nf-core/test-datasets/modules/data/genomics/homo_sapiens/genome/vcf/mills_and_1000G.indels.vcf.gz" \
        --snpeff_db 105 \
        --snpeff_genome "WBcel235" \
        --snpeff_version "5.1" \
        --tools "freebayes" \
        --vep_cache_version "106" \
        --vep_genome "WBcel235" \
        --vep_species "caenorhabditis_elegans" \
        --vep_version "106.1" \
        --max_cpus 4 \
        --max_memory 6.5GB \
        --output "my_results"
        -profile singularity \
        -r 3.2.3
    ```
    
    ??? success "Solution"

        ```json title="my-params.json"
        {
            "igenomes_ignore": true,
            "dbsnp": "https://raw.githubusercontent.com/nf-core/test-datasets/modules/data/genomics/homo_sapiens/genome/vcf/dbsnp_146.hg38.vcf.gz",
            "fasta": "https://raw.githubusercontent.com/nf-core/test-datasets/modules/data/genomics/homo_sapiens/genome/genome.fasta",
            "germline_resource": "https://raw.githubusercontent.com/nf-core/test-datasets/modules/data/genomics/homo_sapiens/genome/vcf/gnomAD.r2.1.1.vcf.gz",
            "intervals": "https://raw.githubusercontent.com/nf-core/test-datasets/modules/data/genomics/homo_sapiens/genome/genome.interval_list",
            "known_indels": "https://raw.githubusercontent.com/nf-core/test-datasets/modules/data/genomics/homo_sapiens/genome/vcf/mills_and_1000G.indels.vcf.gz",
            "snpeff_db": 105,
            "snpeff_genome": "WBcel235",
            "snpeff_version": "5.1",
            "tools":  "freebayes", 
            "vep_cache_version": 106,
            "vep_genome": "WBcel235",
            "vep_species": "caenorhabditis_elegans",
            "vep_version": "106.1",
            "max_cpus": 4,
            "max_memory": "6.5 GB",
            "outdir": "my_results_2"
        }
        ```

        Your execution command will now look like this:

        ```bash
        nextflow run nf-core/sarek --input samplesheet.csv -params-file my-params.json -profile singularity -r 3.2.3
        ```

        Note that in this example we kept `--input samplesheet.csv` in the execution command. However, this could have put this in the `.json` file. You can pick and choose which parameters go in a params file and which parameters go in your execution command.

Due to the order of priority, you can modify parameters you want to change without having to edit your newly created parameters file.

!!! question "Exercise"

    Include both `freebayes` and `strelka` as variant callers using the `tools` parameter and run the pipeline again.

    For this option, you will need to use the `--tools` flag and include both variant callers in the same string separated by a comma, e.g., `--tools "<tool1>,<tool2>"`

    You can also use `-resume` to resume the pipeline from the last successful step.

    ??? success "Solution"

        ```bash
        nextflow run nf-core/sarek --input samplesheet.csv -params-file my-params.json -profile singularity -r 3.2.3 --tools "freebayes,strelka" -resume 
        ```


<br>
!!! circle-info ""

!!! cboard-list-2 "Key points"

    - nf-core pipelines follow a similar structure.
    - nf-core pipelines are configured using multiple configuration sources.
    - Configuration sources are ranked to decide which settings to apply.
    - Pipeline parameters must be passed via the command line (`--<parameter>`) or Nextflow `-params-file` option.
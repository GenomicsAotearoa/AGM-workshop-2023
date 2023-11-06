# Appendix 

## Appendix 1 Shared configuration files

An `includeConfig` statement in the `nextflow.config` file is also used to include custom institutional profiles that have been submitted to the nf-core [config repository](https://github.com/nf-core/configs). At run time, nf-core pipelines will fetch these configuration profiles from the [nf-core config repository](https://github.com/nf-core/configs) and make them available.

For shared resources such as an HPC cluster, you may consider developing a shared institutional profile. You can follow [this tutorial](https://nf-co.re/docs/usage/tutorials/step_by_step_institutional_profile) for more help.


## Appendix 2 Custom config file exercise

!!! question "Exercise"

    Give the MultiQC report for the `christopher-hakkaart/nf-core-demo` pipeline the name of your **favorite food** using the [`multiqc_title`](https://github.com/christopher-hakkaart/nf-core-demo/blob/master/nextflow.config#L27) parameter in a parameters file:

    ??? success "Solution"

        Create a custom `.json` file that contains your favourite food, e.g., cheese:

        ```json title="my-custom-params.json"
        {
        "multiqc_title": "cheese"
        }
        ```

        Include the custom `.json` file in your execution command with the `-params-file` option:

        ```bash
        nextflow run christopher-hakkaart/nf-core-demo -profile test,singularity -r main -params-file my_custom_params.json 
        ```

        Check that it has been applied:

        ```bash
        ls results/multiqc/
        ```

## Appendix 3 Configuration files

Configuration files are `.config` files that can contain various pipeline properties. Custom paths passed in the command-line using the `-c` option:

```bash
nextflow run nf-core/<workflow> -profile test,singularity -c <path/to/custom.config>
```

Multiple custom `.config` files can be included at execution by separating them with a comma (`,`).

Custom configuration files follow the same structure as the configuration file included in the pipeline directory.

Configuration properties are organised into [scopes](https://www.nextflow.io/docs/latest/config.html#config-scopes) by dot prefixing the property names with a scope identifier or grouping the properties in the same scope using the curly brackets notation. For example:

```console title="custom.config"
alpha.x  = 1
alpha.y  = 'string value'
```

Is equivalent to:

```console title="custom.config"
alpha {
     x = 1
     y = 'string value'
}
```

Scopes allow you to quickly configure settings required to deploy a pipeline on different infrastructure using different software management. For example, the `executor` scope can be used to provide settings for the deployment of a pipeline on a HPC cluster. Similarly, the `singularity` scope controls how Singularity containers are executed by Nextflow.

Multiple scopes can be included in the same `.config` file using a mix of dot prefixes and curly brackets. A full list of scopes is described in detail [here](https://www.nextflow.io/docs/latest/config.html#config-scopes).

!!! question "Exercise"

    Give the MultiQC report for the `christopher-hakkaart/nf-core-demo` pipeline the name of your favorite color using the [`multiqc_title`](https://github.com/christopher-hakkaart/nf-core-demo/blob/master/nextflow.config#L27) parameter in a custom `.config` file:

    ??? success "Solution"

        Create a custom `.config` file that contains your favourite colour, e.g., blue:

        ```console title="custom.config"
        params.multiqc_title = "blue"
        ```

        Include the custom `.config` file in your execution command with the `-c` option:

        ```bash
        nextflow run christopher-hakkaart/nf-core-demo -profile test,singularity -r main -resume -c custom.config 
        ```

        Check that it has been applied:

        ```bash
        ls results/multiqc/
        ```

        **Why did this fail?**

        You **can not** use the `params` scope in custom configuration files. Parameters can only be configured using the `-params-file` option and the command line. While it parameter is listed as a parameter on the `STDOUT`, it **was not** applied to the executed command:

        ```bash
        nextflow log
        nextflow log <run name> -f "process,script"
        ```

The `process` scope allows you to configure pipeline processes and is used extensively to define resources and additional arguments for modules.

By default, process resources are allocated in the `conf/base.config` file using the `withLabel` selector:

```bash title="base.config"
process {
    withLabel: BIG_JOB {
        cpus = 16
        memory = 64.GB
    }
}
```

Similarly, the `withName` selector enables the configuration of a process by name. By default, module parameters are defined in the `conf/modules.config` file:

```bash title="modules.config"
process {
    withName: MYPROCESS {
        cpus = 4
        memory = 8.GB
    }
}
```

While some tool arguments are included as a part of a module. To make modules sharable across pipelines, most tool arguments are defined in the `conf/modules.conf` file in the pipeline code under the `ext.args` entry.

Importantly, having these arguments outside of the module also allows them to be customised at runtime.

<br>
<p align="left"><img src="../../images/1_3_args.excalidraw.png" alt="drawing" width="900"/></p> 
<br>

For example, if you were trying to add arguments in the `MULTIQC` process in the `christopher-hakkaart/nf-core-demo` pipeline, you could use the process scope:

```console title="custom.config"
process {
    withName : ".*:MULTIQC" {
        ext.args   = { "<your custom parameter>" }

    }
```

However, if a process is used multiple times in the same pipeline, an extended execution path of the module may be required to make it more specific:

```console title="custom.config"
process {
    withName: "NFCORE_DEMO:DEMO:MULTIQC" {
        ext.args = "<your custom parameter>"
    }
}
```

The extended execution path is built from the pipelines, subworkflows, and module used to execute the process.

In the example above, the nf-core [`MULTIQC`](https://github.com/christopher-hakkaart/nf-core-demo/blob/master/modules/nf-core/multiqc/main.nf) module, was called by the [`DEMO`](https://github.com/christopher-hakkaart/nf-core-demo/blob/master/workflows/demo.nf) pipeline, which was called by the [`NFCORE_DEMO`](https://github.com/christopher-hakkaart/nf-core-demo/blob/master/main.nf) pipeline in the `main.nf` file.

!!! tip "How to build an extended execution path"

    It can be tricky to evaluate the path used to execute a module. If you are unsure of how to build the path you can copy it from the `conf/modules.conf` file. How arguments are added to a process can also vary. Be vigilant.

!!! question "Exercise" 

    Create a new `.config` file that uses the `process` scope to overwrite the `args` for the `MULTIQC` process. Change the `args` to your favourite **month** of the year, e.g, `"--title \"october\""`.

    In this example, the `\` is used to escape the `"` in the string. This is required to ensure the string is passed correctly to the `MULTIQC` module.

    ??? success "Solution"

        Make a custom config file that uses the `process` scope to replace the `args` for the `MULTIQC` process:

        ```console title="custom.config"
        process {
            withName: "NFCORE_DEMO:DEMO:MULTIQC" {
                ext.args = "--title \"october\""
            }
        }
        ```

        Execute your run command again with the custom configuration file:

        ```bash
        nextflow run christopher-hakkaart/nf-core-demo -r main -profile test,singularity -resume -c custom.config
        ```

        Check that it has been applied:

        ```bash
        ls results/multiqc/
        ```

!!! question "Exercise" 

    Demonstrate the configuration hierarchy using the `christopher-hakkaart/nf-core-demo` pipeline by adding a params file (`-params-file`), and a command line flag (`--multiqc_title`) to your execution. You can use the files you have already created. Make sure that the `--multiqc_title` is different to the `multiqc_title` in your params file and different to the title you have used in the past.

    ??? success "Solution"

        Use the `.json` file you created previously:

        ```json title="my-custom-params.json"
        {
        "multiqc_title": "cheese"
        }
        ```

        Execute your command with your params file (`-params-file`) and a command line flag (`--multiqc_title`):

        ```bash
        nextflow run christopher-hakkaart/nf-core-demo -r main -profile test,singularity -resume -params-file my_custom_params.json --multiqc_title "cake"
        ```

        In this example, as the command line is at the top of the hierarchy, the `multiqc_title` will be "cake".


## Configuration files

Sometimes Sarek won't have a parameter that you need to customize the execution of a tool. In this situation, you will need to apply a configuration file to the pipeline.

As shown in the example from [Session 1](../session_1/3_configuration.md), you can selectively apply a configuration file to a process using the `withName` directive.

!!! tip "Be specific with your selector"

    Remember to make your selector specific to the process you are trying to customise. It can be helpful to use the same selectors that are already included in the configuration file.

Sarek is a little different from other pipelines because it has multiple different config files that are used for different tools or groups of tools. This is stylistic and helps to keep the config files organised.

For example, there are a number of options that are applied when calling variants with `FREEBAYES` and are all stored in a [freebayes.config](https://github.com/nf-core/sarek/blob/3.2.3/conf/modules/freebayes.config) file together:

```console title="freebayes.config"
process {

    withName: 'MERGE_FREEBAYES' {
        ext.prefix       = { "${meta.id}.freebayes" }
        publishDir       = [
            mode: params.publish_dir_mode,
            path: { "${params.outdir}/variant_calling/freebayes/${meta.id}/" },
            saveAs: { filename -> filename.equals('versions.yml') ? null : filename }
        ]
    }

    withName: 'FREEBAYES' {
        ext.args         = '--min-alternate-fraction 0.1 --min-mapping-quality 1'
        //To make sure no naming conflicts ensure with module BCFTOOLS_SORT & the naming being correct in the output folder
        ext.prefix       = { meta.num_intervals <= 1 ? "${meta.id}" : "${meta.id}.${target_bed.simpleName}" }
        ext.when         = { params.tools && params.tools.split(',').contains('freebayes') }
        publishDir       = [
            enabled: false
        ]
    }

    withName: 'BCFTOOLS_SORT' {
        ext.prefix       = { meta.num_intervals <= 1 ? meta.id + ".freebayes" : vcf.name - ".vcf" + ".sort" }
        publishDir       = [
            mode: params.publish_dir_mode,
            path: { "${params.outdir}/variant_calling/" },
            pattern: "*vcf.gz",
            saveAs: { meta.num_intervals > 1 ? null : "freebayes/${meta.id}/${it}" }
        ]
    }

    withName : 'TABIX_VC_FREEBAYES' {
        publishDir       = [
            mode: params.publish_dir_mode,
            path: { "${params.outdir}/variant_calling/freebayes/${meta.id}/" },
            saveAs: { filename -> filename.equals('versions.yml') ? null : filename }
        ]
    }

    // PAIR_VARIANT_CALLING
    if (params.tools && params.tools.split(',').contains('freebayes')) {
        withName: '.*:BAM_VARIANT_CALLING_SOMATIC_ALL:BAM_VARIANT_CALLING_FREEBAYES:FREEBAYES' {
            ext.args       = "--pooled-continuous \
                            --pooled-discrete \
                            --genotype-qualities \
                            --report-genotype-likelihood-max \
                            --allele-balance-priors-off \
                            --min-alternate-fraction 0.03 \
                            --min-repeat-entropy 1 \
                            --min-alternate-count 2 "
        }
    }
}
```

Each of these can be modified independently of the others and be applied using a custom configuration file.

!!! question "Exercise"

    Create a custom configuration file that will modify the `--min-alternate-fraction` parameter for `FREEBAYES` to `0.05` and apply it to the pipeline.

    ??? success "Solution"

        ```bash
        nextflow run nf-core/sarek --input samplesheet.csv -params-file my-params.json -profile singularity -r 3.2.3 --tools "freebayes,strelka" -resume -c my-config.config
        ```

        ```console title="my-config.config"
        process {

            withName: 'FREEBAYES' {
                ext.args         = '--min-alternate-fraction 0.05'
            }
        }
        ```

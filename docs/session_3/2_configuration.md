# Finishing touches

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

## Metrics and reports

Nextflow can also produce multiple reports and charts that show several runtime metrics and your execution information. You can enable this functionality by adding Nextflow options to your run command:

- Adding `-with-report` to your run command will create a HTML execution report which includes many useful metrics about a pipeline execution. 
- Adding `-with-trace` option to creates an execution tracing file that contains some useful information about each process executed in your pipeline script.
- Adding `-with-timeline` to your run command enables the creation of the pipeline timeline report showing how processes were executed over time.
- Adding `-with-dag` to your run command enables the rendering of the pipeline execution direct acyclic graph representation.
    - This feature requires the installation of [Graphviz](https://graphviz.org/) on your computer. Beginning in version 22.04, Nextflow can render the DAG as a Mermaid diagram. Mermaid diagrams are particularly useful because they can be embedded in GitHub Flavored Markdown without having to render them yourself.

!!! square-pen "Note" 

    The execution report (`-with-report`), trace report (`-with-trace`), timeline trace (`-with-timeline`), and dag (`-with-dag`) must be specified when the pipeline is executed. By contrast, the `log` option is useful after a pipeline has already run and is available for every executed pipeline.

!!! question "Exercise"

    Try to run the following command and view the reports generated by Nextflow:
    
    ```bash
    nextflow run nf-core/sarek --input samplesheet.csv -params-file my-params.json -profile singularity -r 3.2.3 --tools "freebayes,strelka" -with-report -with-trace -with-timeline -with-dag -resume
    ```

<br>
!!! circle-info ""

!!! cboard-list-2 "Key points"
    
    - Parameters go in parameters files and everything else goes in a configuration file
    - There are additional flags you can use to generate reports and metric for you own records
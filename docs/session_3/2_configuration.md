# Metrics and reports

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
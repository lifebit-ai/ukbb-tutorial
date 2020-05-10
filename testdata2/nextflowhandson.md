## Nextflow hands-on

During this tutorial you will implement a proof of concept of a RNA-Seq pipeline which:
1. Indexes a trascriptome file.
2. Performs quality controls
3. Performs quantification.
4. Create a MultiqQC report.

## Step 1 - define the pipeline parameters

The script ```script1.nf``` defines the pipeline input parameters. Run it by using the following command:
```bash
nextflow run script1.nf
```
Try to specify a different input parameter, for example:
```bash
nextflow run script1.nf --reads this/and/that
```

## Exercise 1.1

Modify the ```script1.nf``` adding a fourth parameter named ```outdir``` and set it to a default path that will be used as the pipeline output directory.

## Exercise 1.2

Modify the ```script1.nf``` to print all the pipeline parameters by using a single ```println``` command and a [multiline](https://www.nextflow.io/docs/latest/script.html#multi-line-strings) string statement.

Tip: see an example [here](https://github.com/nextflow-io/rnaseq-nf/blob/3b5b49f/main.nf#L41-L48).

# Nextflow hands-on

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

### Exercise 1.1

Modify the ```script1.nf``` adding a fourth parameter named ```outdir``` and set it to a default path that will be used as the pipeline output directory.

### Exercise 1.2

Modify the ```script1.nf``` to print all the pipeline parameters by using a single ```println``` command and a [multiline](https://www.nextflow.io/docs/latest/script.html#multi-line-strings) string statement.

Tip: see an example [here](https://github.com/nextflow-io/rnaseq-nf/blob/3b5b49f/main.nf#L41-L48).

### Recap

In this step you have learned:

1. How to define parameters in your pipeline script
2. How to pass parameters by using the command line
3. The use of $var and ${var} variable placeholders
4. How to use multiline strings

## Step 2 - Create transcriptome index file

Nextflow allows the execution of any command or user script by using a ```process``` definition.

A process is defined by providing three main declarations: the process [inputs](https://www.nextflow.io/docs/latest/process.html#inputs), the process [outputs](https://www.nextflow.io/docs/latest/process.html#outputs) and finally the command [script](https://www.nextflow.io/docs/latest/process.html#script).

The second example adds the ```index``` process. Open it to see how the process is defined.

It takes the transcriptome file as input and creates the transcriptome index by using the ```salmon``` tool.

Note how the input declaration defines a ```transcriptome``` variable in the process context that it is used in the command script to reference that file in the Salmon command line.

Try to run it by using the command:
```bash
nextflow run script2.nf
```
The execution will fail because Salmon is not installed in your environment.

Add the command line option ```-with-docker``` to launch the execution through a Docker container as shown below:
```bash
nextflow run script2.nf -with-docker
```


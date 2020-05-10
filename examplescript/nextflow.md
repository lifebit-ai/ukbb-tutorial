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
This time it works because it uses the Docker container ```nextflow/rnaseq-nf``` defined in the ```nextflow.config``` file.

In order to avoid to add the option ```-with-docker``` add the following line in the ```nextflow.config``` file:
```bash
docker.enabled = true
```

### Exercise 2.1
Enable the Docker execution by default adding the above setting in the ```nextflow.config``` file.

### Exercise 2.2
Print the output of the ```index_ch``` channel by using the [println](https://www.nextflow.io/docs/latest/operator.html#println) operator (do not confuse it with the ```println``` statement seen previously).

### Exercise 2.3
Use the command ```tree -a work``` to see how Nextflow organises the process work directory.

### Recap
In this step you have learned:

1. How to define a process executing a custom command
2. How process inputs are declared
3. How process outputs are declared
4. How to access the number of available CPUs
5. How to print the content of a channel

## Step 3 - Collect read files by pairs

This step shows how to match read files into pairs, so they can be mapped by Salmon.

Edit the script ```script3.nf``` and add the following statement as the last line:
```bash
read_pairs_ch.println()
```
Save it and execute it with the following command:
```bash
nextflow run script3.nf
```
It will print an output similar to the one shown below:
```bash
[ggal_gut, [/../data/ggal/gut_1.fq, /../data/ggal/gut_2.fq]]
```
The above example shows how the ```read_pairs_ch``` channel emits tuples composed by two elements, where the first is the read pair prefix and the second is a list representing the actual files.

Try it again specifying different read files by using a glob pattern:
```bash
nextflow run script3.nf --reads 'data/ggal/*_{1,2}.fq'
```

### Exercise 3.1
Use the [set](https://www.nextflow.io/docs/latest/operator.html#set) operator in place of ```=``` assignment to define the ```read_pairs_ch``` channel.

### Exercise 3.2
Use the [ifEmpty](https://www.nextflow.io/docs/latest/operator.html#ifempty) operator to check if the ```read_pairs_ch``` contains at least an item.

### Recap
In this step you have learned:

1. How to use ```fromFilePairs``` to handle read pair files
2. How to use the ```set``` operator to define a new channel variable
3. How to use the ```ifEmpty``` operator to check if a channel is empty

## Step 4 - Perform expression quantification

The script ```script4.nf``` adds the ```quantification``` process.

In this script note as the ```index_ch channel```, declared as output in the ```index``` process, is now used as a channel in the input section.

Also note as the second input is declared as a ```set``` composed by two elements: the ```pair_id``` and the ```reads``` in order to match the structure of the items emitted by the ```read_pairs_ch``` channel.

Execute it by using the following command:
```bash
nextflow run script4.nf 
```
You will see the execution of a ```quantication``` process.

Execute it again adding the ```-resume``` option as shown below:
```bash
nextflow run script4.nf -resume 
```
The ```-resume``` option skips the execution of any step that has been processed in a previous execution.

Try to execute it with more read files as shown below:
```bash
nextflow run script4.nf -resume --reads 'data/ggal/*_{1,2}.fq'
```
You will notice that the ```quantification``` process is executed more than one time.

Nextflow parallelizes the execution of your pipeline simply by providing multiple input data to your script.

### Exercise 4.1
Add a [tag](https://www.nextflow.io/docs/latest/process.html#tag) directive to the ```quantification``` process to provide a more readable execution log .

### Exercise 4.2
Add a [publishDir](https://www.nextflow.io/docs/latest/process.html#publishdir) directive to the ```quantification``` process to store the process results into a directory of your choice.

### Recap
In this step you have learned:

1. How to connect two processes by using the channel declarations
2. How to resume the script execution skipping already already computed steps
3. How to use the ```tag``` directive to provide a more readable execution output
4. How to use the ```publishDir``` to store a process results in a path of your choice

## Step 5 - Quality control

This step implements a quality control of your input reads. The inputs are the same read pairs which are provided to the ```quantification``` steps

You can run it by using the following command:
```bash
nextflow run script5.nf -resume 
```
The script will report the following error message:
```bash
Channel `read_pairs_ch` has been used twice as an input by process `fastqc` and process `quantification`
```

### Exercise 5.1
Modify the creation of the ```read_pairs_ch``` channel by using a [into](https://www.nextflow.io/docs/latest/operator.html#into) operator in place of a ```set```.

Tip: see an example [here](https://github.com/nextflow-io/rnaseq-nf/blob/3b5b49f/main.nf#L58).

### Recap
In this step you have learned:

How to use the ```into``` operator to create multiple copies of the same channel

## Step 6 - MultiQC report

This step collect the outputs from the ```quantification``` and ```fastqc``` steps to create a final report by using the [MultiQC](https://multiqc.info/) tool.
```bash
nextflow run script6.nf -resume --reads 'data/ggal/*_{1,2}.fq' 
```
It creates the final report in the ```results``` folder in the current work directory.

Execute the script with the following command:
```bash
nextflow run script6.nf -resume --reads 'data/ggal/*_{1,2}.fq' 
```

In this script note the use of the [mix](https://www.nextflow.io/docs/latest/operator.html#mix) and [collect](https://www.nextflow.io/docs/latest/operator.html#collect) operators chained together to get all the outputs of the ```quantification``` and ```fastqc``` process as a single input.

### Recap
In this step you have learned:

1. How to collect many outputs to a single input with the ```collect``` operator
2. How to ```mix``` two channels in a single channel
3. How to chain two or more operators togethers

## Step 7 - Handle completion event

This step shows how to execute an action when the pipeline completes the execution.

Note that Nextflow processes define the execution of asynchronous tasks i.e. they are not executed one after another as they are written in the pipeline script as it would happen in a common imperative programming language.

The script uses the ```workflow.onComplete``` event handler to print a confirmation message when the script completes.

Try to run it by using the following command:
```bash
nextflow run script7.nf -resume --reads 'data/ggal/*_{1,2}.fq' 
```



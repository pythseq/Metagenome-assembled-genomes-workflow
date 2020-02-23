# Metagenomic analysis README
## Description:
The scripts available here follow along the following steps, - coassembly or cross assembly of all the reads using 1)SPAdes 2)MEGAHIT - assembly statistics using 1 )QUAST 2) Bowtie2 - Binning similar contigs toegther to 
reconstruct genomes, 1) MetaBat - Bin qulaity statistics using 1) CheckM, Kraken, Centrifuge, BUSCO - Taxa annotation after each steps using 1)Kraken, 2)Centrifuge, 3)FOCUS - Functional annotation for the bins using 1)SUPEFOCUS 
These scripts will help you get started with the basic steps employed in metagenomic analysis, starting with taxa/functional annotation of the intial reads and assembled contigs. If you are insterested in further reconstructing 
the genomes from the metagenomes, you can continue running the binning steps to tie the different function back to taxa. This is by no means the end of the ananlysis steps, these are the general steps, the next further steps will 
vary base don your research questions.

## Steps to run the scripts

**SETTING UP SCRIPTS FIRST** 
1. Run the following commands, that will find the job scripts saved with an extension .sh and replaces the emailaddress and adds the path
        make sure to CHNAGE MY EMAIL ADDRESS TO YOURS HERE,
        for f in */*.sh; do sed -i 's/YOUREMAILHERE/email@iu.edu/g' $f; done
        for f in */*.sh; do p=`pwd`; sed -i "s|PWDHERE|$p|g" $f ; done 

LETS START WITH THE READS** 
Make sure all the reads do end with the extension "*.fastq" and not "*.fq".
2. Add you reads as files to the reads directory. In the reads directory, run the command

        cat *1.fastq >left.fq
        cat *2.fastq >right.fq 

This command joins all the left reads(ending with 1.fastq) toegther to left.fq and all the right reads (ending with 2.fastq). 

**ASSEMBLY AND ASSEMBLY REPORTS** 
3. Go to to assembly directory to start assembling the reads. The script co-assembles all the samples together, and individual assemblies for each sample. 
Before you start, take a look at the job script and make sure the email and path is set correctly before you submit the job. 
To take a look at the jobs script you can use the command

        less spades.sh
        less megahit.sh 

To run the job script, the command is
        
        qusb spades.sh
        qsub megahit.sh 

Wait for these jobs to complete. Take a look at the job logs before you continue. 
OUTPUT - saved to a directory called contigs- spades_output, sample-name_spades_output, megahit_output, and sample-name_megahit_output

4. Deduplicate the assembled contigs from all the assemblies to remove the exact duplicates using bbtools, dedupe.sh command. To run this step, 
submit the job. 

	qsub dedupe.sh

OUTPUT - saved to directory called contigs - dedupe_contig.fa
 
5. Once the assembly jobs are completed, then run the next script. 
        
        qsub quast.sh 

The quast.sh script runs assembly statistics to all the assemblers (coassembly, individual assemblies, deduplicated contigs), and writes the output to a table.
OUTPUT- saved to a file called quast_table.tsv. 
Take a look at the quast_table.tsv, pick the assembler that likely produced the most number of reads, with high N50 lengths. 

6. Move the asemmbled contigs you would like to use for the rest of the workflow from the contigs directory. 
Generally since dedupe.sh would have more contigs since it contains the final deduplicated set of contigs from the assemblies. 
Do make note that using this result for estimating taxa/functional abundance can highly bias the results. 
	mv assembly/contigs/dedupe.sh assembly/final_contigs.fa

**BINNING AND BIN QUALITY REPORTS** 
7. Lets start grouping similar sequences together now. The script binning.sh runs metabat on both the spades and megahit assembly, check to see the script looks right before you submit the job
        
        qsub bamfiles.sh

Wait for this job to complete before starting the next one. 

	qsub metabat2.sh
	qsub concoct.sh

OUTPUT - MetaBat2 outputs the bins to metabat_bins and concoct outputs to concoct_bins. Both these directories should have a list of bins that was put from the two programs. 

8. Running dastool that takes the bins from metabat2 and concoct to remove non-redundant set of bins from both the binning algorithms. This will generate one set of outputs 
	
	qsub dastool.sh

############################################################################################ 
9. Check bin quality of the bins Next once the bins are generated, run the next script to calculate the completeness/taxa for each bins. Run the script spades_bin_quality.sh
        
        qsub spades_bin_quality.sh 

If you are interested in running the scripts for metabat as well, here are the steps -Copy the job script file with a new name
        
        cp spades_bin_quality.sh metabat_bin_quality.sh 

In the script, you will want to replace the directory names from spades_ *to megahit_*. You can do this manually, or run the following commands
        
        sed -i 's/spades/megahit/g' megahit_bin_quality.sh 

WARNING:The sed command only works if your files have the same convention as spades bin directory. For example, in this case, the bins from spades assembly is saved to 
spades_metabat, and bins from megahit is saved to megahit_metabat.
#################################################################################################

**TAXA ANNOTATION** 
Run three different taxa annotation tools - FOCUS, Kraken2 and Centrifuge on the reads 

	qsub kraken.sh 
Output is saved to the file "kraken-report-final.csv"

	qsub focus.sh 
Output is saved to the files in a directory "focus_output/".

	qsub centrifuge.sh 
Output is saved to multiple files. 

**FUNCTIONAL ANNOTATION**
## Contact
Email bhnala@iu.edu or help@ncgas.org

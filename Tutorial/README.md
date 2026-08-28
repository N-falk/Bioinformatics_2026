# Tutorial: Merging, Quality Control, Filtering & Alignment

In this tutorial, we will work through some of the basic steps involved in processing short-read sequencing data.

We will start with paired-end FASTQ files and work through:

1. Downloading and preparing sequencing data
2. Assessing sequence quality using **FastQC**
3. Merging paired-end reads using **fastp**
4. Performing quality control on the merged reads
5. Filtering reads based on quality and length
6. Aligning reads to a reference genome using **Minimap2**
7. Processing alignment files using **Samtools**
8. Interpreting basic alignment statistics

The assignment is divided into two parts:

* **Part A — Merging, Quality Control and Filtering**
* **Part B — Alignment**

---

# Part A — Merging, Quality Control and Filtering

## Step 1: Download and Prepare the Data

### 1. Log in to DeepThought

Open your Ubuntu Virtual Desktop and use JupyterLab Web to open a **Terminal**.

Log in to DeepThought using SSH:

```bash
ssh YOURFAN@deepthought.flinders.edu.au
```

Replace `YOURFAN` with your Flinders FAN.

Once logged in, navigate to your scratch directory:

```bash
cd /scratch/user/$USER
```

`$USER` is an environment variable that automatically contains your username/FAN.

You can check where you are using:

```bash
pwd
```

---

### 2. Create a directory for this tutorial

Create a new directory for this practical and move into it:

```bash
mkdir friday_tutorial
cd friday_tutorial
```

You can check the contents of your directory using:

```bash
ls -lh
```

---

### 3. Download the sequencing dataset

The dataset contains **8 FASTQ files**, representing four sequencing samples.

Download the dataset using `wget`:

```bash
wget https://zenodo.org/record/1236641/files/test_fastq_small.zip
```
If you get a response such as 'command not found', try running 'wget --version' to see if it's installed.

You should now have a file called:

```text
test_fastq_small.zip
```

---

### 4. Unzip the dataset

Use `unzip` to extract the FASTQ files:

```bash
unzip test_fastq_small.zip
```

Check what has been created:

```bash
ls -lh
```

The eight FASTQ files represent four samples.

Each sample contains two files:

* **R1** — forward reads
* **R2** — reverse reads

For example:

```text
Test01_L001_R1_001.fastq
Test01_L001_R2_001.fastq
```

These are **paired-end sequencing reads**. Each DNA fragment has been sequenced from both ends, producing a forward and reverse read.

---

# Step 2: Initial Quality Control with FastQC

Before modifying our sequencing data, we should first examine its quality.

We will use **FastQC** to generate a quality-control report.

### 5. Activate the FastQC environment

You should already have a Conda environment containing FastQC from the previous practical.

Activate it:

```bash
conda activate fastqc_env
```

Check that FastQC is available:

```bash
fastqc --help
```

---

### 6. Run FastQC on the Test01 reads

For this exercise, we will initially examine the forward and reverse reads from Test01.

Run:

```bash
fastqc Test01_L001_R1_001.fastq
```

and:

```bash
fastqc Test01_L001_R2_001.fastq
```

This will generate HTML reports containing information about the quality of the sequencing data.

You should now see files similar to:

```text
Test01_L001_R1_001_fastqc.html
Test01_L001_R2_001_fastqc.html
```

---

### 7. View the FastQC reports

You can view the reports from the command line using `lynx`:

```bash
lynx Test01_L001_R1_001_fastqc.html
```

and:

```bash
lynx Test01_L001_R2_001_fastqc.html
```

You can navigate through the report using the arrow keys and press `q` to exit.

You can also copy the HTML files back to your JupyterLab environment using `scp` and view them in a normal web browser, as demonstrated in the previous practical.

---

### 1

From the FastQC reports, inspect the **Basic Statistics** section for both files.
Take note of:

* Number of sequences
* Sequence length
* Total sequence output
* Any other obvious differences between R1 and R2

---

# Step 3: Merge the Paired-End Reads

We will now use **fastp** to merge the paired-end reads.

The idea is that the R1 and R2 reads originate from opposite ends of the same DNA fragment. If the reads overlap, the overlapping information can be used to reconstruct a longer sequence.

### 8. Activate the fastp environment

Deactivate FastQC:

```bash
conda deactivate
```

Then activate the fastp environment:

```bash
conda activate fastp_env
```

Check that fastp is available:

```bash
fastp --help
```

---

### 9. Merge Test01 R1 and R2

Run the following command:

```bash
fastp \
  -i Test01_L001_R1_001.fastq \
  -I Test01_L001_R2_001.fastq \
  --merge \
  --merged_out Test01_merged.fastq \
  --disable_adapter_trimming \
  --disable_quality_filtering \
  --html fastp_merge_report.html \
  --json fastp_merge_report.json
```

The important parts of this command are:

```text
-i
```

Specifies the R1/forward FASTQ file.

```text
-I
```

Specifies the R2/reverse FASTQ file.

```text
--merge
```

Tells fastp to attempt to merge paired-end reads.

```text
--merged_out
```

Specifies the output file containing the merged reads.

```text
--disable_adapter_trimming
```

Prevents adapter trimming at this stage.

```text
--disable_quality_filtering
```

Prevents quality filtering at this stage.

We are deliberately separating **merging** from **quality filtering** so that we can examine what happens at each stage.

The command also creates HTML and JSON reports:

```text
fastp_merge_report.html
fastp_merge_report.json
```

---

# Step 4: Quality Control After Merging

We can now examine the merged sequences using FastQC.

### 10. Run FastQC on the merged reads

First activate the FastQC environment:

```bash
conda deactivate
conda activate fastqc_env
```

Run FastQC:

```bash
fastqc Test01_merged.fastq
```

This will generate:

```text
Test01_merged_fastqc.html
```

View the report using:

```bash
lynx Test01_merged_fastqc.html
```

---

### Question 3 — What happened to the sequence length?

Compare the sequence length of:

* The original R1 reads
* The original R2 reads
* The merged reads

Are the sequences in the merged FASTQ file **longer or shorter** than the original R1 and R2 reads?

Explain why this might be the case.

Think about how paired-end sequencing works and what happens when two overlapping reads are combined.

---

# Step 5: Repeat the Merging for All Samples

We have only merged Test01 so far.

Repeat the process for:

* Test02
* Test03
* Test04

Remember to change the input and output filenames for each sample.

For example, Test02 would use:

```bash
fastp \
  -i Test02_L001_R1_001.fastq \
  -I Test02_L001_R2_001.fastq \
  --merge \
  --merged_out Test02_merged.fastq \
  --disable_adapter_trimming \
  --disable_quality_filtering \
  --html Test02_fastp_merge_report.html \
  --json Test02_fastp_merge_report.json
```

Do the same for Test03 and Test04.

At the end of this step, you should have four merged FASTQ files:

```text
Test01_merged.fastq
Test02_merged.fastq
Test03_merged.fastq
Test04_merged.fastq
```

---

# Step 6: Filter the Merged Reads

We will now filter the merged reads based on sequence quality and length.

In the previous practical, we used fastp to remove low-quality sequence and bases from the beginning of reads.

For this exercise, use the following filtering thresholds:

* **Quality threshold:** `-q 30`
* **Minimum length:** `-f 30`

The `-q 30` option sets the minimum base quality score to Q30.

The `-f 30` option removes the first 30 bases from each read.

Use the fastp filtering command from the [Bioinformatics 2026 GitHub tutorial](https://github.com/N-falk/Bioinformatics_2026).

You will need to apply the filtering to all four merged FASTQ files.

For example, your Test01 command could look like:

```bash
fastp \
  -i Test01_merged.fastq \
  -o Test01_merged_filtered.fastq \
  -q 30 \
  -f 30
```

Repeat this for Test02, Test03 and Test04, changing the input and output filenames appropriately.

---

### Question 4 — Quality control before and after filtering

Run FastQC on the filtered Test01 FASTQ file.

For example:

```bash
fastqc Test01_merged_filtered.fastq
```

Open the resulting report:

```bash
lynx Test01_merged_filtered_fastqc.html
```

Compare the **Basic Statistics** section with the FastQC report from before filtering.

Consider:

* Has the number of sequences changed?
* Has the sequence length changed?
* Has the total sequence output changed?
* Why would filtering produce these changes?

---

# Part B — Alignment

In Part B, we will take our processed sequencing reads and align them against a known reference genome.

This is an important concept in bioinformatics.

Instead of simply asking:

> "What sequences are in my FASTQ file?"

we can ask:

> "Where do these sequences match a known genome?"

We will use:

* **Minimap2** — to align the reads
* **Samtools** — to process and summarise the alignments

---

# Step 7: Set Up the Alignment Exercise

### 11. Obtain the Assignment 1 files

The files required for the alignment exercise are available in the Assignment 1 directory of the 2025 course GitHub repository:

```bash
git clone https://github.com/N-falk/Bioinformatics_2025
```

Then navigate to the Assignment 1 directory:

```bash
cd Bioinformatics_2025/Assignment1
```

Use:

```bash
ls -lh
```

to see the available files.

---

## 12. Install Minimap2

Minimap2 is a sequence alignment program designed for mapping DNA or RNA sequences to reference sequences.

Create a conda environment for minimap2 and activate it. 

```bash
conda create --name minimap2_env
```

Activate it:

```bash
conda activate minimap2_env
```

Download and extract Minimap2:

```bash
curl -L https://github.com/lh3/minimap2/releases/download/v2.31/minimap2-2.31_x64-linux.tar.bz2 | tar -jxvf -
./minimap2-2.31_x64-linux/minimap2
```

Test the installation:

```bash
./minimap2-2.31_x64-linux/minimap2
```

You should see information about Minimap2 and its available options.

---

## 13. Install Samtools

We will use Samtools to process the alignment files produced by Minimap2.

Create a Conda environment for the alignment tools:

```bash
conda create --name alignment_env
```

Activate it:

```bash
conda activate alignment_env
```

Install Samtools:

```bash
conda install -c bioconda samtools
```

Check that Samtools is installed:

```bash
samtools --help
```

---

# Step 8: Index the Reference Genome

Before aligning reads, we first need to create an index of the reference sequence.

The reference genome is contained in a FASTA file.

For example:

```text
reference.fasta
```

Minimap2 can create an index using:

```bash
./minimap2-2.31_x64-linuxminimap2 -d ref.mmi reference.fasta
```

This creates:

```text
ref.mmi
```

The `.mmi` file is an index of the reference sequence that Minimap2 can use to perform alignments efficiently.

---

### — What is a reference genome?

When aligning DNA sequences, what is the purpose of the **reference genome**?

Consider why we need a known sequence to compare our sequencing reads against.

---

# Step 9: Align Reads to the Reference

We can now align our sequencing reads to the reference genome.

The following command uses Minimap2 to align the reads and pipes the output directly into Samtools for sorting:

```bash
minimap2 -ax sr ref.mmi sample_merged.fastq.gz | samtools sort -o merged.sorted.bam
```

There are several things happening in this command.

### Minimap2

```bash
minimap2 -ax sr ref.mmi sample_merged.fastq.gz
```

This aligns the FASTQ reads against the indexed reference.

The `-ax sr` option specifies a short-read alignment preset.

### Pipe

```text
|
```

The pipe sends the output from Minimap2 directly into the next program rather than creating an intermediate file.

### Samtools

```bash
samtools sort -o merged.sorted.bam
```

Samtools takes the alignment output and sorts it, producing a BAM file.

The resulting file is:

```text
merged.sorted.bam
```

---

## 14. Index the BAM file

Once the BAM file has been sorted, create an index:

```bash
samtools index merged.sorted.bam
```

This creates:

```text
merged.sorted.bam.bai
```

The BAM index allows programs to quickly access particular regions of the alignment file.

---

## 15. Calculate alignment statistics

Use `samtools flagstat` to summarise the alignment:

```bash
samtools flagstat merged.sorted.bam
```

This will provide information including:

* Number of reads
* Number of mapped reads
* Percentage of reads mapped
* Other alignment statistics

---

### SAM vs BAM

Consider the difference between **SAM** and **BAM** file formats.

Consider:

* What does SAM stand for?
* What does BAM stand for?
* How are the two formats related?
* Why might BAM be preferable for storing large sequencing datasets?

---

### Question 7 — Test01 alignment

Using the output from:

```bash
samtools flagstat merged.sorted.bam
```

Consider the the following:

1. What percentage of reads from **Test01** mapped to the reference?
2. What might this result indicate about the Test01 sample?

---

# Step 10: Align *E. coli* Reads

For the final part of the exercise, we will use a FASTQ file containing *E. coli* sequencing reads.

The file is:

```text
ecoli_1K_2.fq.00.0_0.cor.fast.gz
```

We will align these reads to the same reference genome.

---

## 16. Align the *E. coli* reads

Run:

```bash
minimap2 -ax sr ref.mmi ecoli_1K_2.fq.00.0_0.cor.fast.gz | samtools sort -o ecoli.sorted.bam
```

Then index the BAM file:

```bash
samtools index ecoli.sorted.bam
```

Finally, calculate the alignment statistics:

```bash
samtools flagstat ecoli.sorted.bam
```

---

### Question 8 — *E. coli* alignment

Using the `samtools flagstat` output:

1. What percentage of reads from the *E. coli* FASTQ file mapped to the reference?
2. What might this indicate about the relationship between the *E. coli* sequences and the reference genome?

Compare your answer with the Test01 alignment.

---

# Final Workflow

By completing this assignment, you have worked through a simplified sequencing data-processing pipeline:

```text
             Paired-end FASTQ files
                       │
                       ▼
                    FastQC
                       │
                       ▼
             Initial quality check
                       │
                       ▼
                Merge R1 + R2
                   (fastp)
                       │
                       ▼
                 Merged reads
                       │
                       ▼
                    FastQC
                       │
                       ▼
               Quality filtering
                   (fastp)
                       │
                       ▼
                Filtered reads
                       │
                       ▼
              Reference genome
                       │
                       ▼
                 Minimap2
                       │
                       ▼
                  Alignment
                       │
                       ▼
               Sorted BAM file
                 (Samtools)
                       │
                       ▼
              Alignment statistics
              (samtools flagstat)
```

## Tools used

| Tool       | Purpose                                                       |
| ---------- | ------------------------------------------------------------- |
| `wget`     | Download sequencing data                                      |
| `unzip`    | Extract compressed files                                      |
| `Conda`    | Manage software environments                                  |
| `FastQC`   | Assess sequencing quality                                     |
| `fastp`    | Merge and filter sequencing reads                             |
| `Minimap2` | Align reads to a reference genome                             |
| `Samtools` | Sort, index and summarise alignments                          |
| `lynx`     | View HTML reports from the command line                       |
| `scp`      | Transfer files between DeepThought and your local environment |

## Learning outcomes

After completing this assignment, you should be able to:

* Recognise the structure of paired-end sequencing data.
* Explain the difference between R1 and R2 reads.
* Perform basic quality control using FastQC.
* Explain why paired-end reads can be merged.
* Merge sequencing reads using fastp.
* Apply quality and length filtering to FASTQ files.
* Compare sequencing data before and after filtering.
* Explain the purpose of a reference genome.
* Align sequencing reads to a reference using Minimap2.
* Explain the difference between SAM and BAM files.
* Use Samtools to process and summarise alignment files.
* Interpret the percentage of reads that map to a reference genome.

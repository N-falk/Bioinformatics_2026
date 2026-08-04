# Bioinformatics 2026: DeepThought Access & FASTQ Processing Exercises

---

## Logging into Deepthought via JupyterHub

Open your browser and go to:  
[http://deepteachweb.flinders.edu.au/jupyter](http://deepteachweb.flinders.edu.au/jupyter)

By default, when you log in, you will be placed in your **home directory**.  
However, we recommend working in the **scratch directory** because it has more storage space.

---

## Linking Scratch Directory in Jupyter File Browser

Create a symbolic link in your home directory to your scratch directory to make it easier to navigate:

```bash
ln -s /scratch/user/$USER scratch
```
Now the scratch directory will appear in your Jupyter file browser.
To undo/remove this link without deleting your actual scratch files, use:

```bash
rm scratch
```
Note:
This only deletes the symbolic link, not the original files.

To delete a file: rm file_name

To delete a directory and its contents: rm -rf directory_name

## Exercise: Testing subsamp.py and count_and_qual.py

Step 1: Create a directory in your scratch for the test files

```bash
cd /scratch/user/USERNAME
mkdir directory_name
cd directory_name
```
Alternatively, navigate in the Jupyter file browser to your scratch folder and create a directory.

Step 2: Obtain the files uing Git

The files are located on Github at https://github.com/N-falk/Bioinformatics_2025, and we will use the Git command to copy them to your scratch directory on Deepthought.

Check if git is installed:

```bash
git version
```

If not installed, on systems where you have permissions:

```bash
sudo apt update
sudo apt install git
```
Clone the repository containing the files:

```bash
git clone https://github.com/N-falk/Bioinformatics_2025
```

Step 3: Navigate to the fastq_fun directory and prepare files

```bash
cd Bioinformatics_2025/fastq_fun
ls -lh
gunzip *.fastq.gz
```
This unzips all .fastq.gz files to .fastq.

Step 4: Prepare Python environment and scripts
Check Python version:

```bash
python --version
```
Install Biopython (required by the Python scripts):

```bash
pip install biopython
```

Convert .txt scripts to .py files for execution. You can view the .txt versions of the script, but let's make them .py files first:

```bash
nano count_and_qual.py
# Paste contents of count_and_qual.txt, then save (Ctrl+X, Y, Enter)

nano subsamp.py
# Paste contents of subsamp.txt, then save
```

For better understanding, copy-paste the script contents into your preferred AI assistant and ask about each part of the code.

Step 5: Run the Python scripts

```bash
python count_and_qual.py
python subsamp.py
```

It is more efficient for deepthought to make a bash script so that we can run these python codes as submitted jobs to dedicated nodes, and deepthought's slurm manager can take car ot it.
Open the file subsamp.slurm, and copy and paste in the following, starting with the line #!/bin/bash and ending with the line python subsamp.py:

```bash
#!/bin/bash

#SBATCH --job-name=subsamp_USERNAME
#SBATCH --output=%x-%j.out.txt
#SBATCH --error=%x-%j.err.txt
#SBATCH --partition=high-capacity
#SBATCH --qos=hc-concurrent-jobs
#SBATCH --time=4-0
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=10G

python subsamp.py
```
Submit the job to Deepthought using the "sbatch" command:

```bash
sbatch subsamp.slurm
```

Check job status:

```bash
squeue --me
```

Repeat the same process for count_and_qual.py using a count_and_qual.slurm file with this content:

```bash
#!/bin/bash

#SBATCH --job-name=countqual_USERNAME
#SBATCH --output=%x-%j.out.txt
#SBATCH --error=%x-%j.err.txt
#SBATCH --partition=high-capacity
#SBATCH --qos=hc-concurrent-jobs
#SBATCH --time=4-0
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=10G

python count_and_qual.py
```

Submit and monitor similarly:

```bash
sbatch count_and_qual.slurm
squeue --me
```

## FastQC and Fastp Exercises

Navigate to the directory FastQC_fastp that's in Bioinformatics_2025. You can try this on your own or use the code below from wherever you are, replacing USER with your user FAN, and make sure you are in the correct relative path. Note that your file path structure may be different depending on where you've put things and what you've called them :)

```bash
cd /scratch/user/USER/Bioinformatics_2025/FastQC_fastp
```
We will use conda to install and manage program installations. Conda is a package manager, and helps partition programs and installations and keeps them nice and tidy, without interference from other programs. Check the conda version that is (hopefully) installed on Deepthought:

```bash
conda --version
```
If you get "command not found", it means conda isn’t installed or isn’t in your PATH. On an HPC, you might need to first load it:

```bash
module load Miniconda3
```

```bash
conda init bash
```
Logout of the terminal and start a new one for the changes to take place.

Create a conda environment to install fastqc:

```bash
conda create --name fastqc_env
```
Type y (as in yes) to accept the install.

If you need to remove a conda environment use:

```bash
conda env remove --name myenv
```
Activate the fastqc conda environment:

```bash
conda activate fastqc_env
```
You can see a list of all your conda environments by using conda env list

### Install FastQC 

With your environment set up, install FastQC by running:

```bash
conda install -c bioconda fastqc
```
Verify the installation by executing:

```bash
fastqc --help
```
You should see a list of the fastQC instructions/options.

### Run fastqc

```bash
fastqc -o ./ ./*.fastq
```

This command will run fastqc and generate QC reports for all .fastq files in your current location (that's what the * notation followed by a file type will do). Replace *.fastq with the a specific file name to run an individual file.

You could make this a .slurm script to submit fastqc as a job, following the same instructions as previously for subsamp.slurm or count_and_qual.slurm

Deactivate the fastqc environment:

```bash
conda deactivate
```

### Install Fastp

Create a new conda environment to install fastp and then activate it:

```bash
conda create --name fastp_env
conda activate fastp_env
```
Install fastp:

```bash
conda install -c bioconda fastp
```
Verify the installation by executing:

```bash
fastp --help
```

### Run fastp on example fatsq files in /FastQC_fastp

```bash
fastp -i Sample1.fastq -o Sample1_fastp.fastq -q 27 -f 30
```

This code takes Sample1.fastq and makes a cleaned version called Sample1_fastp.fastq, then it chops off the first 30 bases of every read (maybe because of adapters or poor-quality sequence at the start).
Then, at both read ends, trims bases with a quality score below Q27 (~0.2% error rate). Can you modify the script to run on all .fastq files in the directory?

You can make this a .slurm script to submit fastqc as a job, following the same instructions as above for subsamp.slurm or count_and_qual.slurm.

Activate the fastqc_env environment again, and run fastqc on the file you just created with fastp:


```bash
fastqc -o ./ ./Sample1_fastp.fastq
```
Compare the fastqc reports (the .html files) for before and after applying fastp filtering to Sample1; you'll notice that fastp also outputs a small report (the fastp.html file)

### Side Quest

Instead of using fastqc, try and install and run Multiqc: 

https://github.com/MultiQC/MultiQC

# Bioinformatics 2026: DeepThought Access & FASTQ Processing Exercises

---

## Logging into DeepThought via Ubuntu Virtual Desktop 

Open your Flinders Okta Dashboard and select the VirtualApps Portal.
From here, find the "Ubuntu Virtual Desktop" portal under "Desktops".
This will open a version of the Linux operating system Ubuntu, complete with several applications:



<img width="653" height="319" alt="image" src="https://github.com/user-attachments/assets/ccdae887-ff72-46e6-83e3-168df8bf6ba4" />


We will use a command line interface to access and talk to DeepThought from your device.
In Ubuntu Virtual Desktop, there is an application called JupyterLab Web, and one called Terminal. Both utilise a command line interface that we can use to connect to DeepThought.

Although you can use either, I recommend JupyterLab Web, as the copy and paste functions in the terminal window are easier and more reliable across operating systems.

Once you open JupyterLab Web, the launcher will have a "Terminal" option. Select this, and a command line terminal interface will open. You will see a flashing cursor next to your flinders FAN username.
This is where we will connect to DeepThought using the secure shell (ssh) protocol. To connect, type ssh followed by YOURFAN@deepthought.flinders.edu.au, replacing YOURFAN with your Flinders FAN id.

For example, if your FAN is smit0028, enter: ssh smit0028@deepthought.flinders.edu.au and then hit enter.

You may be prompted to type "yes" to an agreement, followed by your password. Enter your password at the flashing cursor and hit enter (and recall, you won't be able to see your password being entered, but it is being detected).

When first connected to DeepThought, you are interacting with the "head node".
The head node acts as the central gateway where users log in to write scripts, compile code, and submit tasks to a job scheduler (i.e., slurm).

---

## Locating and moving to your scratch directory on DeepThought

By default, when you log in to DeepThought, you will be placed in your **home directory**.
You can see this using the pwd (print working directory) command.
However, we recommend working in the **scratch directory** because it has more storage space.

Using the cd (change directory) command, navigate to your **scratch directory**:

```bash
cd /scratch/user/$USER
```
$USER is your FAN id

Now if you use pwd, you will see that you are in **scratch**.


## Exercise 1: Testing subsamp.py and count_and_qual.py

Step 1: Create a directory in your scratch for the test files

```bash
cd /scratch/user/USERNAME
mkdir directory_name
cd directory_name
```
Alternatively, navigate in the Jupyter file browser to your scratch folder and create a directory.

Step 2: Obtain the files uing Git

The files are located on Github at https://github.com/N-falk/Bioinformatics_2026, and we will use the Git command to copy them to your scratch directory on Deepthought.

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
git clone https://github.com/N-falk/Bioinformatics_2026
```

Step 3: Navigate to the fastq_fun directory and prepare files

```bash
cd Bioinformatics_2026/fastq_fun
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

There are several .txt files that contain python code we will use. To view these .txt files in command line, use 'less file_name.txt'
To close the file, press 'q' (to quit).

Convert .txt scripts to .py files for execution. You can view the .txt versions of the script, but let's make them .py files first.
We will use the command line text editor "nano" to open a new file, copy and paste in some python code, then save it as a ".py" python script file:

```bash
nano count_and_qual.py
# Paste contents of count_and_qual.txt, then save (Ctrl+X, Y, Enter)

nano subsamp.py
# Paste contents of subsamp.txt, then save
```

For better understanding, copy-paste the script contents into your preferred AI assistant and ask about each part of the code.

Step 5: Run the Python scripts

Python code can be run on the DeepThought headnode by typing "python" followed by the name of the .py script

```bash
python count_and_qual.py
python subsamp.py
```
*The next section is an example of how to submit and run these python scripts as jobs on DeepThought using bash scripts and slurm.*

It is more efficient for deepthought to make a bash script so that we can run these python codes as submitted jobs to dedicated nodes, and deepthought's slurm manager can take care of it.
Open the file subsamp.slurm using nano, and copy and paste in the following, starting with the line #!/bin/bash and ending with the line python subsamp.py:

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
You could then submit the job to Deepthought using the "sbatch" command:

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
In this scenario I have provided blank .slurm files for you. However, you can create your own .slurm files using the text editor nano, for example.
(In this case, you can use the command "nano count_and_qual.slurm", which creates and opens an editable text file, from which you can copy and paste in the bash script. As before, 
you can save nano files by typing "Ctrl+X", then "Y", then "Enter"). 

## Exercise 2: FastQC and Fastp Exercises

Navigate to the directory FastQC_fastp that's in Bioinformatics_2026. You can try this on your own or use the code below from wherever you are, replacing USER with your user FAN, and make sure you are in the correct relative path. Note that your file path structure may be different depending on where you've put things and what you've called them :)

```bash
cd /scratch/user/USER/directory_name/Bioinformatics_2026/FastQC_fastp
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
You can see a list of all your conda environments by using 'conda env list'

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
Compare the fastqc reports (the .html files) for before and after applying fastp filtering to Sample1; you'll notice that fastp also outputs a small report (the fastp.html file).

The generated .html file is a more user friendly output to view the fastq file parameters. You can use the following command to view the file in command line:

```bash
lynx filename.html
```

Running this code will open a text-like viewing file in command line where you can navigate with the arrow keys through the file. However, it is more intuitive to view the .html report file outside of command line.
So we will save the .html file to your local space on the JupyterLab App and then view it from there. We will copy the file to your local computer using the "scp" (secure copy protocol) command, which is a common way to
copy files from an HPC.

Open a new terminal window in JupyterLab. Without logging in to DeepThought, use the following command to copy a file from your DeepThought location to your JupyterLab location:

```bash
scp USER@deepthought.flinders.edu.au:/scratch/user/USER/directory_name/Bioinformatics_2026/FastQC_fastp/Sample1_fastqc.html .
```
In this instance, replace USER with your FAN, and make sure the file path matches your location on DeepThought where the .html file is. The period (.) at the end of the command indicates that you want to copy the .html file to your current directory on JupyterLab, which is probably /home/ISD/USER. IF the scp command works, you should then see the .html file appear in your list of files in the left side window in JupyterLab (after refreshing). You can open the .html file now and see it in all its glory!

### Side Quest

Instead of using fastqc, try and install and run Multiqc: 

https://github.com/MultiQC/MultiQC

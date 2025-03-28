# LARgeno
State of the art KIR3DL1 genotyping currently relies on a multiplex PCR, amplifying multiple regions of the KIR3DL1 locus. The resulting sequences then have to be puzzled together to derive the KIR3DL1 genotype. 

LARgeno introduces a streamlined approach: a single DNA amplification, followed by sequencing with Oxford Nanopore technology. The generated sequences are then analyzed using the LARgeno pipeline, which provides high-confidence KIR3DL1 genotyping and a reference-based sequence assembly.

## Installation 
The LARgeno bash script relies on two conda enviroments (LARgeno and Clair3) and thus either anaconda or miniconda. 

### 1. Install Miniconda/Anaconda  

Before installing LARgeno, ensure **Miniconda** or **Anaconda** is installed on your system:  

- **Miniconda:** [Download Here](https://www.anaconda.com/docs/getting-started/miniconda/install)  
- **Anaconda:** [Download Here](https://www.anaconda.com/products/distribution)  

Once installed, restart your terminal to apply changes.  

### 2. Install LARgeno Environment  

Create and configure the **LARgeno** environment:  
```bash
conda create --name LARgeno r-essentials r-base samtools=1.13
conda activate LARgeno
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```
Install required bioinformatics tools:

```bash
conda install bioconda::minimap2 bioconda::bcftools bioconda::seqtk
pip3 install --user whatshap
conda install bioconductor-VarCon r-seqinr bioconductor-DECIPHER r-vcfR r-data.table
```

### 3. Install Clair3 Environment  

Create and activate the **Clair3** environment:
```bash
conda deactivate
conda create -n clair3 python=3.9.0 -y
conda activate clair3

```
Install PyPy and required Python packages:
```bash
conda install -c conda-forge pypy3.6 -y
pypy3 -m ensurepip
pypy3 -m pip install mpmath==1.2.1
```
Install TensorFlow and additional dependencies:
```bash
conda install -c conda-forge tensorflow==2.8.0 -y
conda install -c conda-forge pytables -y
conda install -c anaconda pigz cffi==1.14.4 -y
conda install -c conda-forge parallel=20191122 zstd -y
conda install -c conda-forge -c bioconda samtools=1.15.1 -y
conda install -c conda-forge -c bioconda whatshap=1.7 -y
conda install -c conda-forge xz zlib bzip2 automake curl -y
# tensorflow-addons is required in training
pip install tensorflow-addons
```
Clone the Clair3 repository and compile necessary tools:

```bash
git clone https://github.com/HKU-BAL/Clair3.git
cd Clair3

# compile samtools, longphase and cffi library for c implement
# after building, longphase binary is in `Clair3` folder
conda activate all && make PREFIX=${CONDA_PREFIX}
```
Download pre-trained Clair3 models:
```bash
mkdir models
wget http://www.bio8.cs.hku.hk/clair3/clair3_models/clair3_models.tar.gz 
tar -zxvf clair3_models.tar.gz -C ./models
```
Install additional tools:
```bash
conda install samtools bcftools
conda install bioconda::minimap2
pip install numpy==1.26.4
```

### 4. Download LARgeno gitub repository
Finally download the LARgeno gitub repository containing scripts and configuration files
```bash
git clone https://github.com/caggtaagtat/LARgeno.git
```
## Usage
### Generatl Usage
```bash
bash ./LARgeno.sh \
-i $INPUTPATH \
-o $OUTPUTPATH \
-m $MINICONDAPATH \
-g $GUPPYPATH \
-c $CLAIRPATH \
-r $REFERENCE \
-b $REFERENCE_BACKGROUND \
-l $MINLENGTH \
-t $THREADS 
```
### Options
#### Required parameters
```bash
-i         Input path. Path to the directory of the sequencing data (if in tar.gz compressed format, provied path of tar.gz file). Must contain sub-folder "fastq_pass". (Example: "/home/user/genoKIR/KIRPool2" or "/home/user/genoKIR/KIRPool2.tar.gz")
-o         Output path. Path where the output directory $INPUTPATH"genotyp" will be generated. (Example: "/home/user/genoKIR/")
-m         Path to miniconda or anaconda directory. (Example: "/home/user/miniconda3/")
-g         Path to guppy bin directory (please use the last guppy version 6.5.7. (Example: "/home/coronam/bin/ont-guppy-6.5.7/bin/" )
-c         Path to Clair3 directory. (Example: "/home-ssd/jopto100/software/Clair3")
-r         Reference sequences as FASTA file. (Example: "/home/user/references/IPDKIR_release_2.14.fasta")
-l         Minimal Read length treshold. (Example: "10000")
-t         Maximal number of threads to use. (Example: "20")
```

#### Optional parameters
```bash
-b        Reference background FASTA file. Reads will be aligned against both the reference and the reference background file, keeping only reads that best align against the reference. (Example: /home/user/references/References_except_targetGene_IPDKIR_release_2.14.fasta")
```

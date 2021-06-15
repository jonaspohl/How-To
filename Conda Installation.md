## Installing Conda
We prefer to use Conda package manager to install various packages on Linux. Run `conda` command in the terminal to check whether conda is installed. If not, please follow these steps:

###  max-cluster:
1. Install conda by running:
```
  env GUIX_PACKAGE_PATH=guix-conda/ guix package -i conda 
```

2. Paste the output of this command into your ~/.bashrc file (otherwise conda activate will not work):  
```
conda info | grep -i 'base environment' | awk '{print  “/etc/profile.d/conda.sh”}' 
```

3. Reload ~/.bashrc by running:
 `
source ~/.bashrc 
`

### Mac
Install the latest version of anaconda: https://www.anaconda.com/distribution/.

### Configuring Conda
```  
conda config --add channels defaults 
conda config --add channels bioconda 
conda config --add channels conda-forge 
```

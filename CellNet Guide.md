## Installation

  

1. Install conda
[Conda Installation Guide](https://github.com/MDC-Berlin-Kaminski-team/How-To/blob/main/Conda%20Installation.md)
  

2. Install the following packages under conda:

    `conda create -n cellnet_env salmon=0.8.2 cutadapt parallel python=3.6 tbb=2020.2=hc9558a2_0`

      Note: do not install the latest versions of salmon, python or tbb. They will not work with cellnet.


3. Activate conda environment:

     `conda activate cellnet_env`


## Running

### Human example from

[https://www.cahanlab.org/resources/nprot.2017.022_full.pdf](https://www.cahanlab.org/resources/nprot.2017.022_full.pdf)


#### step 2
```
library(CellNet)
cn_setup(local=TRUE)
```
  

#### step3

```
iFileHuman <- "salmon.index.human.052617.tgz"
fetchIndexHandler(destination="ref/", species="human", iFile=iFileHuman)
```
  

#### step 4 (download the annotation table)

```
download.file("https://s3.amazonaws.com/CellNet/rna_seq/human/examples/SRP043684/st_SRP043684_example.rda",
  "st_SRP043684_example.rda")
stQuery<-utils_loadObject("st_SRP043684_example.rda")
```
  
#### step 5

```
stQuery<-cn_s3_fetchFastq("CellNet",
  "rna_seq/human/examples/SRP043684", stQuery, fname="fname", compressed="gz")
save(stQuery, file="stQuery.rda")
```
  

#### step 6

```
iFileHuman <- "hs_index_v3"
pathToSalmon <-"~/anaconda3/envs/cellnet_env/bin/"
load("stQuery.rda")
expList<-cn_salmon(stQuery,refDir="ref/", salmonIndex=iFileHuman,
  geneTabfname="geneToTrans_Homo_sapiens.GRCh38.80.exo_Jul_04_2015.R",
  salmonPath=pathToSalmon)
save(expList, file="expList.rda")
```
  

#### step 7

```
download.file(
  "https://s3.amazonaws.com/CellNet/rna_seq/human/cnProc_RS_hs_Oct_25_2016.rda",
  dest="./cnProc.rda")
```
  

#### step 8

```
load("expList.rda")
cnProc<-utils_loadObject("cnProc.rda")
cnRes<-cn_apply(expList[['normalized']], stQuery, cnProc)
save(cnRes, file="cnRes.rda")
```
  

#### step 9

```
load("cnRes.rda")
pdf(file='hmclass.pdf', width=7, height=5)
cn_HmClass(cnRes)
dev.off()
```
  

#### step 10

```
load("stQuery.rda")
load("cnRes.rda")
cnProc<-utils_loadObject("cnProc.rda")
fname<-'grnstats_esc_subset_SRP043684.pdf'
bOrder<-c("esc_train", unique(as.vector(stQuery$description2)), "neuron_train")
cn_barplot_grnSing(cnRes, cnProc,"esc", c("esc", "neuron"),
   bOrder, sidCol="sra_id", dlevel="description2")
ggplot2::ggsave(fname, width=5.5, height=5)
dev.off()
fname<-'grnstats_neuron_subset_SRP043684.pdf'
bOrder<-c("esc_train", unique(as.vector(stQuery$description2)), "neuron_train")
cn_barplot_grnSing(cnRes, cnProc,"neuron", c("esc", "neuron"),
   bOrder, sidCol= "sra_id", dlevel='description2')
ggplot2::ggsave(fname, width=5.5, height=5)
dev.off()
````
  

#### step 11

```
load("stQuery.rda")
load("cnRes.rda")
cnProc<-utils_loadObject("cnProc.rda")
rownames(stQuery)<-as.vector(stQuery$sra_id)
tfScores<-cn_nis_all(cnRes, cnProc, "neuron")
fname='nis_neuron_subset_example_ctrlipsNeurons.pdf'
plot_nis(tfScores, "neuron", stQuery, "Control iPS neurons", dLevel="description2", limitTo=0)
ggplot2::ggsave(fname, width=4, height=12)
dev.off()
```

### Running Michael's mouse data

1.  Create a directory where for the specific data set, e.g. `CellNet_Hnf1b_mouse` and `cd` there.  
    
2.  Create `CellNetLocal` directory and copy all the files from Michael's data set directory, in particular: `*.fastq`, `sampTabFileName.csv` 
    
3.  Launch `R` from `CellNet_Hnf1b_mouse directory` and run the following commands:
  

#### step 2

```
library(CellNet)
cn_setup(local=TRUE)
```
  

#### step 3

```
fetchIndexHandler(destination="ref/", species="mouse", iFile= "salmon.index.mouse.052617.tgz")
```

#### no need in steps 4-5

#### step 6

```
stQuery=read.csv("sampTabFileName.csv")
pathToSalmon <-"~/anaconda3/envs/cellnet_env/bin/"
expList<-cn_salmon(stQuery,refDir="ref/", salmonIndex="mm_index_052617", salmonPath=pathToSalmon)
save(expList, file="expList.rda")
```

#### step 7

```
download.file(
  "https://s3.amazonaws.com/CellNet/rna_seq/mouse/cnProc_MM_RS_Oct_24_2016.rda", dest="./cnProc.rda")
```

#### step 8

```
load("expList.rda")
cnProc<-utils_loadObject("cnProc.rda")
stQuery=read.csv("sampTabFileName.csv")
cnRes<-cn_apply(expList[['normalized']], stQuery, cnProc)
save(cnRes, file="cnRes.rda")
```

#### step 9

```
load("cnRes.rda")
pdf(file='hmclass.pdf', width=7, height=5)
cn_HmClass(cnRes)
dev.off()
```

#### step 10

```
stQuery=read.csv("sampTabFileName.csv")
load("cnRes.rda")
cnProc<-utils_loadObject("cnProc.rda")
fname<- 'grnstats_fibroblast_example.pdf'
bOrder<- c("fibroblast_train", unique(as.vector(stQuery$description1)),"kidney_train")
cn_barplot_grnSing(cnRes,cnProc,"fibroblast", c("fibroblast","kidney"), bOrder, sidCol="sra_id")
ggplot2::ggsave(fname, width=5.5, height=5)
dev.off()
fname<- 'grnstats_kidney_example.pdf'
bOrder<- c("fibroblast_train", unique(as.vector(stQuery$description1)),"kidney_train")
cn_barplot_grnSing(cnRes,cnProc,"kidney", c("fibroblast","kidney"), bOrder, sidCol="sra_id")
ggplot2::ggsave(fname, width=5.5, height=5)
dev.off()
```

#### step 11 (modified)

```
stQuery=read.csv("sampTabFileName.csv")
load("cnRes.rda")
cnProc<-utils_loadObject("cnProc.rda")
rownames(stQuery)<-as.vector(stQuery$sra_id)
tfScores<-cn_nis_all(cnRes, cnProc, "**kidney**")
for (exp in unique(stQuery$description1)) {
  plot_nis(tfScores, "kidney", stQuery, exp, dLevel="description1", limitTo=0)
  ggplot2::ggsave(paste0(exp, ".pdf"), width=8, height=12)
}
```

### Running Michael's human data

1.  Create a directory where for the specific data set, e.g. `CellNet_Human` and `cd` there.  
    
2.  Create `CellNetLocal` directory and copy all the files from Michael's data set directory, in particular: `*.fastq`, `sampTabFileName.csv`  
    
3.  Launch `R` from `CellNet_Human directory` and run the following commands:

#### step 2

```
library(CellNet)
cn_setup(local=TRUE)
```

#### step 3

```
fetchIndexHandler(destination="ref/", species="human", iFile= "salmon.index.human.052617.tgz")
```

#### no need in steps 4-5

#### step 6

```
stQuery=read.csv("sampTabFileName.csv")
pathToSalmon <-"~/anaconda3/envs/cellnet_env/bin/"
expList<-cn_salmon(stQuery,refDir="ref/", salmonIndex="hs_index_v3",
  geneTabfname="geneToTrans_Homo_sapiens.GRCh38.80.exo_Jul_04_2015.R",
  salmonPath=pathToSalmon)
save(expList, file="expList.rda")
```

#### step 7

```
download.file(
  "https://s3.amazonaws.com/CellNet/rna_seq/human/cnProc_RS_hs_Oct_25_2016.rda", dest="./cnProc.rda")
```

#### step 8

```
load("expList.rda")
cnProc<-utils_loadObject("cnProc.rda")
stQuery=read.csv("sampTabFileName.csv")
cnRes<-cn_apply(expList[['normalized']], stQuery, cnProc)
save(cnRes, file="cnRes.rda")
```

#### step 9

```
load("cnRes.rda")
pdf(file='hmclass.pdf', width=7, height=5)
cn_HmClass(cnRes)
dev.off()
```


#### preparation for step 10:

CellNet has a bug in cn_extract_SN_DF function => replace it

```
fixed.cn_extract_SN_DF<-function(scores, sampTab, dLevel, rnames=NULL, sidCol="sample_id") {
  if(is.null(rnames))
    rnames<-rownames(scores);

  tss<-scores[rnames,];
  if(length(rnames)==1){
    tss<-t(as.matrix(scores[rnames,]));
    rownames(tss)<-rnames;
  }
  colnames(tss) <- colnames(scores); # Misha: bug fix!
  nSamples<-ncol(tss);
  stTmp<-sampTab[colnames(tss),]; ###
  snNames<-rownames(tss);
  num_subnets<-length(snNames);

  snNames<-unlist(lapply(snNames, rep, times=nSamples));
  sample_ids<-rep(as.vector(stTmp[,sidCol]), num_subnets);
  descriptions<-rep(as.vector(stTmp[,dLevel]), num_subnets);
  scores<-as.vector(t(tss));
  data.frame(sample_id=sample_ids, description=descriptions, subNet = snNames, score=scores);
}

assignInNamespace("cn_extract_SN_DF", fixed.cn_extract_SN_DF, ns="CellNet")
```

#### step 10

```
stQuery=read.csv("sampTabFileName.csv")
load("cnRes.rda")
cnProc<-utils_loadObject("cnProc.rda")
fname<- 'grnstats_fibroblast_example.pdf'
bOrder<- c("fibroblast_train", unique(as.vector(stQuery$description1)), "kidney_train")
cn_barplot_grnSing(cnRes,cnProc,"fibroblast", c("fibroblast","kidney"), bOrder, sidCol="sra_id")
ggplot2::ggsave(fname, width=5.5, height=5)
dev.off()
fname<- 'grnstats_kidney_example.pdf'
bOrder<- c("fibroblast_train", unique(as.vector(stQuery$description1)),"kidney_train")
cn_barplot_grnSing(cnRes,cnProc,"kidney", c("fibroblast","kidney"), bOrder, sidCol="sra_id")
ggplot2::ggsave(fname, width=5.5, height=5)
dev.off()
```

#### step 11 (modified)

```
stQuery=read.csv("sampTabFileName.csv")
load("cnRes.rda")
cnProc<-utils_loadObject("cnProc.rda")
rownames(stQuery)<-as.vector(stQuery$sra_id)
tfScores<-cn_nis_all(cnRes, cnProc, "**kidney**")
for (exp in unique(stQuery$description1)) {
  plot_nis(tfScores, "kidney", stQuery, exp, dLevel="description1", limitTo=0)
  ggplot2::ggsave(paste0(exp, ".pdf"), width=8, height=12)
}
```
  

## Resources

[https://github.com/pcahan1/CellNet#bulk_protocol](https://github.com/pcahan1/CellNet#bulk_protocol)

[https://www.cahanlab.org/resources/nprot.2017.022_full.pdf](https://www.cahanlab.org/resources/nprot.2017.022_full.pdf)

[https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE168620](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE168620)

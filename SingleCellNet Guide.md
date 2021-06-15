## Installation

```
install.packages('png')
install.packages("devtools")
devtools::install_github("pcahan1/singleCellNet")
```

## Running on aquarius

In the Web Browser go to: https://aquarius.mdc-berlin.net/rstudio-sys/

In R run:

```
library(singleCellNet)
setwd('~/project/scn/example')

# Load query data
stPark = utils_loadObject("sampTab_Park_MouseKidney_062118.rda")
expPark = utils_loadObject("expMatrix_Park_MouseKidney_Oct_12_2018.rda")
genesPark = rownames(expPark)
rm(expPark)
gc()

# load the training data
expTMraw = utils_loadObject("expMatrix_TM_Raw_Oct_12_2018.rda")
stTM = utils_loadObject("sampTab_TM_053018.rda")
stTM<-droplevels(stTM)

# Find genes in common to the data sets and limit analysis to these
commonGenes = intersect(rownames(expTMraw), genesPark)
expTMraw = expTMraw[commonGenes,]

# Split for training and assessment, and transform training data
set.seed(100) #can be any random seed number
stList = splitCommon(sampTab=stTM, ncells=100, dLevel="newAnn")
stTrain = stList[[1]]
expTrain = expTMraw[,rownames(stTrain)]

# Train the classifier
if (file.exists("class_info.rda")) {
    load("class_info.rda")
} else {
    system.time(class_info<-scn_train(stTrain = stTrain, expTrain = expTrain,
      nTopGenes = 10, nRand = 70, nTrees = 1000, nTopGenePairs = 25, dLevel = "newAnn", colName_samp = "cell"))
    save(class_info, file = "class_info.rda")
}

# Apply to held out data
#validate data
#normalize validation data so that the assessment is as fair as possible:
stTestList = splitCommon(sampTab=stList[[2]], ncells=100, dLevel="newAnn")
stTest = stTestList[[1]]
expTest = expTMraw[commonGenes,rownames(stTest)]
#predict
classRes_val_all = scn_predict(cnProc=class_info[['cnProc']], expDat=expTest, nrand = 50)

# Assess classifier
tm_heldoutassessment = assess_comm(ct_scores = classRes_val_all, stTrain = stTrain,
  stQuery = stTest, dLevelSID = "cell", classTrain = "newAnn", classQuery = "newAnn", nRand = 50)
plot_PRs(tm_heldoutassessment)

# Plot metrics
plot_metrics(tm_heldoutassessment)

# Classification result heatmap
#Create a name vector label used later in classification heatmap where the values are
#cell types/ clusters and names are the sample names
 
nrand = 50
sla = as.vector(stTest$newAnn)
names(sla) = as.vector(stTest$cell)
slaRand = rep("rand", nrand) 
names(slaRand) = paste("rand_", 1:nrand, sep='')
sla = append(sla, slaRand) #include in the random cells profile created

sc_hmClass(classMat = classRes_val_all,grps = sla, max=300, isBig=TRUE)

# Attribution plot
plot_attr(classRes=classRes_val_all, sampTab=stTest, nrand=nrand, dLevel="newAnn", sid="cell")

# Viusalize average top pairs genes expression for training data
gpTab = compareGenePairs(query_exp = expTest, training_exp = expTrain, training_st = stTrain,
  classCol = "newAnn", sampleCol = "cell", RF_classifier = class_info$cnProc$classifier, numPairs = 20,
  trainingOnly= TRUE)

train = findAvgLabel(gpTab = gpTab, stTrain = stTrain, dLevel = "newAnn")

hm_gpa_sel(gpTab, genes = class_info$cnProc$xpairs, grps = train, maxPerGrp = 50)

# Apply to Park et al query data
expPark = utils_loadObject("expMatrix_Park_MouseKidney_Oct_12_2018.rda") 
  
nqRand = 50
system.time(crParkall<-scn_predict(class_info[['cnProc']], expPark, nrand=nqRand))

# Visualization
sgrp = as.vector(stPark$description1)
names(sgrp) = as.vector(stPark$sample_name)
grpRand =rep("rand", nqRand)
names(grpRand) = paste("rand_", 1:nqRand, sep='')
sgrp = append(sgrp, grpRand)

# heatmap classification result
sc_hmClass(crParkall, sgrp, max=5000, isBig=TRUE, cCol=F, font=8)

# Classification annotation assignment
# This classifies a cell with  the catgory with the highest classification score or
# higher than a classification score threshold of your choosing.
# The annotation result can be found in a column named category in the query sample table.

stPark <- get_cate(classRes = crParkall, sampTab = stPark, dLevel = "description1",
  sid = "sample_name", nrand = nqRand)

# Classification result violin plot
sc_violinClass(sampTab = stPark, classRes = crParkall, sid = "sample_name", dLevel = "description1", addRand = nqRand)

# Skyline plot of classification results
library(viridis)
stKid2 = addRandToSampTab(crParkall, stPark, "description1", "sample_name")
skylineClass(crParkall, "T cell", stKid2, "description1",.25, "sample_name")
```

## Resources

https://github.com/pcahan1/singleCellNet

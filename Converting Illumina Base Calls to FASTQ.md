## Installation
 
1.  On Windows machine install Illumina Experiment Manager:  
    [https://support.illumina.com/sequencing/sequencing_software/experiment_manager/downloads.html](https://support.illumina.com/sequencing/sequencing_software/experiment_manager/downloads.html)
2.  On Linux / Mac: install bcl2fastq:  
    conda install -c dranew bcl2fastq

## Conversion

1.  Create a Sample Sheet (the experiment annotation file that is needed for further conversion) in Illumina Experiment Manager. Name it SampleSheet.csv. More about it:  
    [https://bioinformatics.cvr.ac.uk/how-to-demultiplex-illumina-data-and-generate-fastq-files-using-bcl2fastq/](https://bioinformatics.cvr.ac.uk/how-to-demultiplex-illumina-data-and-generate-fastq-files-using-bcl2fastq/)
2.  Copy the SampleSheet.csv into the Illumina Base Call directory, e.g. 210528_MN00157_0085_A000H3GH5Y.
3.  Run the conversion:  
    ```
    nohup bcl2fastq --runfolder-dir 210528_MN00157_0085_A000H3GH5Y -p 12 --output-dir fastq_files --no-lane-splitting &
    ```
      
    Remarks:

-   `-p 12` indicates 12 threads being used for the conversion process. If this parameter is omitted, the conversion will use all the available CPU cores.
-   The conversion process may run long. `nohup` ensures the command continues running even when the user session is finished (e.g. the user has logged out).

## Resources

https://emea.support.illumina.com/content/dam/illumina-support/documents/documentation/software_documentation/bcl2fastq/bcl2fastq_letterbooklet_15038058brpmi.pdf

[https://bioinformatics.cvr.ac.uk/how-to-demultiplex-illumina-data-and-generate-fastq-files-using-bcl2fastq/](https://bioinformatics.cvr.ac.uk/how-to-demultiplex-illumina-data-and-generate-fastq-files-using-bcl2fastq/)

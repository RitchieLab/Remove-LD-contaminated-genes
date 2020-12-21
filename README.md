This code can conduct statistical colocalization between GWAS-significant SNPs and the corresponding eQTLs (for a given tissue) on a gene-by-gene basis. This is ideally suited for situations in which we have a set of genes and tissues of interest (e.g. obtained from running Transcriptome wide Association Studies) and corresponding gene-expression and GWAS summary statistics.

There is no required version of R, but R (>=3.5.0) is preferred.

The following libraries are required to run this sofware:
* [coloc](https://cran.r-project.org/web/packages/coloc/index.html)
* [simsalapar](https://cran.r-project.org/web/packages/simsalapar/index.html)
* [optparse](https://www.rdocumentation.org/packages/optparse/versions/1.6.6)
* [data.table](https://cran.r-project.org/web/packages/data.table/data.table.pdf)
* Installed [GCTA](https://cnsgenomics.com/software/gcta/#Overview) v1.26 or higher

### Coloc protocol:
* For running *colocalization*, we first identify all the SNPs in the GWAS dataset that are within a specified window (default=1Mb) from the TSS and TES of the selected gene for a given tissue. Of these, we consider as “lead SNPs” all SNPs in the GWAS dataset with a p-value < a chosen threshold that are > 100 KB apart from each other. 
* We collect the SNPs overlapping between GWAS and eQTL datasets for each lead SNP-eGene pair for a given tissue. 
* We subsequently estimate the coloc probability of H3 (alternative hypothesis that eQTL and GWAS associations correspond to independent signals) and H4 (alternative hypothesis that eQTL and GWAS associations correspond to the same signal) for the lead SNP-eGene pair. 
* We assume a prior probability that a SNP is associated with (1) lipid phenotype (default=1E-04), (2) gene expression (default=1E=0-4), and (3) both GWAS and gene expression (default=1E-06) for all coloc analyses. 
* We select the lead SNP with the lowest P[H3] for a given eGene and use this as the corresponding P[H3] for the gene. 
* We can subsequently filter out all genes whose P[H3]>0.5 for a given tissue (these could be LD contaminated; i.e. GWAS causal variant and eQTL are different but in LD). 

### GCTA-COJO protocol: 
* We use the following GCTA-COJO protocol in the GWAS and eQTL datasets for each lead SNP to identify putative independent seecondary associations at a locus. 
* For each lead SNP, we obtain p-values conditional on the top-associated eQTL at that locus for GWAS and eQTL datasets using GCTA-COJO. 

```
#sample GWAS file
SNP	BP	CHR	A1	A2	BETA	SE	P	FREQ	N
rs1172982	100230111	1	t	c	0.0043	0.0055	0.4689	0.3219	89888
rs1172981	100230197	1	t	c	0.0057	0.0103	0.7688	0.06069	89888
rs11166327	100230867	1	c	t	0.0053	0.0106	0.7725	0.06069	89888
rs4908018	100234367	1	a	c	0.0058	0.0103	0.7887	0.05937	89841
rs2392072	100234743	1	a	g	0.0039	0.0058	0.4052	0.3153	86877
```

```
#sample eQTL file
gene_id	variant_id	tss_distance	ma_samples	ma_count	maf	pval_nominal	slope	slope_se
ENSG00000261456.5	chr10_11501_C_A_b38	-62662	124	124	0.116105	0.0400165	-0.187313	0.0909802
ENSG00000261456.5	chr10_11553_G_C_b38	-62610	108	108	0.105058	0.0216029	-0.221475	0.0961113
ENSG00000261456.5	chr10_18924_A_C_b38	-55239	86	86	0.0776173	0.283069	-0.114165	0.106242
ENSG00000261456.5	chr10_44215_T_TTCTG_b38	-29948	13	13	0.0122411	0.608588	0.130235	0.254164
ENSG00000261456.5	chr10_45349_G_A_b38	-28814	21	21	0.0180723	0.579648	0.124365	0.224383
```

#### GCTA steps
* We use the --cojo-cond option to perform model selection and get a list of independently associated testable SNPs (p-value < chosen threshold). 
* In our example dataset, we use 1000 genome EUR (chromosome 1) as reference dataset to calculate pairwise LD. 


## Setup and Example

1. Clone the repository.
``` 
git clone https://github.com/RitchieLab/Gene-level-statistical-colocalization
```
2. Go to the cloned folder (set as working directory).
``` 
cd Gene-level-statistical-colocalization 
```
3. Download and unzip example data https://ritchielab.org/files/Lipid_Pleiotropy_project/LD_Contamination_example_data.tar.gz
and save downloaded data under the cloned folder.
4. Add ```gcta64``` to the folder path.

5. Run ```run_gcta_and_coloc.R```.
```
Rscript run_gcta_and_coloc.R \
--gwas_data_name=${dataset} \
--trait=${trait} \
--tissue=${tissue} \
--gene_of_interest=${ensg} \
--cojo_maf=0.01 \
--chr=${chr} \
--coloc_p1=1e-04 \
--coloc_p2=1e-04 \
--coloc_p12=1e-06 \
--lead_snp_window_size=100000 \
--gene_boundary_window_size=1000000 \
--gwas_p_threshold=0.0001 \
--eqtl_p_threshold=0.001 \
--core=10 \
--gwas_response_type="cc" \
--gwas_file=${path_gwas}/${dataset_gwas}_${trait}.txt \
--eqtl_file=${path_gtex}/${tissue}.allpairs.chr${chr}.txt.gz \
--genes_file=${genes_file} \
--output_folder=${output_folder} \
--reference_folder=${reference_folder} \
--lead_snps_file=${lead_snps} \
--liftover_filename=hg38ToHg19.over.chain.gz \
--eqtl_sample_size=${num_eqtl_file}"
```

Following is an explanation of the listed parameters:

  * --*chromosome* The chromosome to which the given gene(s) correspond(s) 
  * --*trait* Name of the trait 
  * --*tissue* Name of tissue corresponding to the eQTL dataset 
  * --*gwas_data_name* Name of GWAS dataset (e.g. GLGC, GIANT)
  * --*gene_of_interest* ENSG_gene(s) of interest from genes_file (ignore decimal point)
  * --*gene_boundary_window_size* The window around the chosen gene from the TSS and TES of the gene (default = 1 Mb) 
  * --*lead_snp_window_size* The window around the lead SNP used for colocalization analysis
  * --*gwas_p_threshold* The threshold for GWAS p-value for SNP variants corresponding to the chosen gene(s) (default = 1E-03) 
  * --*eqtl_p_threshold* The threshold for eQTL p-value for SNP variants mapping to the chosen eGene(s) (default = 1E-02)
  * --*gwas_response_type* The response variable type for GWAS (quant or cc)
  * --*eqtl_file* File for eQTL summary statistics (from GTEx) for chosen tissue, split by chromosome, with path 
  * --*coloc_p1* Prior probability a SNP is associated with GWAS trait (default = 1E-04)
  * --*coloc_p2* Prior probability a SNP is associated with gene expression (default = 0.001) 
  * --*coloc_p12* Prior probability a SNP is associated with GWAS trait and gene expression (default = 1E-06)
  * --*gwas_file* File for tab separated GWAS summary statistics data (with header) for chosen trait with column names SNP, BP, CHR, A1, A2, BETA, SE, P, FREQ, with path 
  * --*genes_file* File (with path) for tab separated list of chosen genes with column names =  ENSG_gene, gene_start_position, gene_stop_position, chromosome
  * --*lead_snps_file* External Input file of GWAS lead varID, i.e. chr:bp
  * --*cojo_maf* MAF threshold used in GCTA-COJO(default=0.01) 
  * --*cojo_p* P-value threshold for gcta-cojo (default = 1E-03)
  * --*reference_folder* Path of the folder with plink files for LD calculation in gcta (should have chromosome number in the filename in .chromosome.bim/bed/fam format)
  * --*output_folder* Path of the output folder
  * --*core* Number of cores to run parallel tasks (default = 10) 
  * --*ld_folder* Path of the folder with plink files for LD calculation in gcta (should have chromosome number in the filename in .chromosome.bim/bed/fam format)
  * --*eqtl_sample_size* Filename (with path) of sample sizes for eQTL datasets across different tissues; has two columns corresponding to tissue name and sample size
  * --*liftover_filename* Filename (with path) to UCSC chainfiles for liftover from GRCh 38 in GTEx v8 to GRCh 37
  
6. Run ```remove_non_colocalized_genes.R```
```
Rscript remove_non_colocalized_genes.R \
  --file_name chr1_GLGC_LDL_Adipose_Subcutaneous_colocProbs.txt \
  --coloc_p_h3_threshold 0.5 \
  --coloc_p_h4_threshold 0.01 \
  --output_folder "output"
 ```
 
Following is an explanation of the listed parameters:

  * --*file_name* Load coloc output file obtained after running run_gcta_and_coloc.R
  * --*coloc_p_h3_threshold* Threshold for minP[H3] per gene across all lead SNPs in the gene (default=0.5)
  * --*coloc_p_h4_threshold* Threshold for maxP[H4] per gene across all lead SNPs in the gene (default=0.01)
  * --*output_folder* Folder where list of genes after filtering out ones with no evidence of colocalization with GWAS SNPs is saved
  
## Reference
The manuscript "Unified framework identifies novel replicating links between plasma lipids and diseases from Electronic Health Records across large-scale cohorts" is currently under review.

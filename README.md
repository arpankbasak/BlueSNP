Copyright IBM Corp. 2011, 2012

BlueSNP
==========

### R package for highly scalable genome wide association studies using Hadoop clusters

News
----------

* 10/29/2012 BlueSNP package and documentation posted. Source will be posted shortly. Contact rjprill@us.ibm.com for more information.

Getting started
----------

Regular users who are not interested in the source code should download the following files from [Downloads](https://github.com/ibm-bioinformatics/BlueSNP/downloads).
* BlueSNP R package
* BlueSNP Manual (installation instructions)
* BlueSNP Tutorial (usage instructions)
* Tutorial data (for following along)

BlueSNP_0.1.0 (current version) depends on [RHIPE_0.69](https://github.com/saptarshiguha/RHIPE/downloads). Because RHIPE installation is [non-trivial](https://www.datadr.org/install.html), we recommend trying BlueSNP using the [RHIPE virtual machine (VM)](https://docs.google.com/open?id=0BzruSBxuthmjUS1vU3lzOENXWlU) made available by the authors of RHIPE. The BlueSNP Manual gives step-by-step instructions for using the RHIPE VM with BlueSNP.

Synopsis
----------

### Analyze many phenotypes (e.g., diseases)

    library(BlueSNP)                         # GWAS functions using Hadoop

Import SNP data in [PLINK tped format](http://pngu.mgh.harvard.edu/~purcell/plink/data.shtml#tr).

    read.plink.tped(
      input.hdfs.path="/tped",               # where tped file(s) are located
      output.hdfs.path="/snps"               # where BlueSNP records will be written
    )

Analyze two phenotypes reporting only those SNPs with small p-values.

    gwas(
      genotype.hdfs.path="/snps",            # input
      phenotype.hdfs.path="/pheno.RData",    # input
      output.hdfs.path="/output",            # output
      phenotype.cols=c("pheno1", "pheno2"),  # selected phenotypes
      method="qt.linear.regression",         # association test
      pvalue.report.cutoff=1e-5              # only report SNPs with small p-values
    )
    
    P = gwas.results("/output", type="p.value")    

Output:

          type       rsid chr        bp       pheno1       pheno2
    1  p.value  rs1898285   3 161531378 6.104797e-08          NaN
    2  p.value rs10936248   3 161536436 6.701088e-08          NaN
    3  p.value  rs4604153   5  26231934 4.240296e-07          NaN
    4  p.value  rs6779918   3 161540640 1.274853e-06          NaN
    5  p.value  rs1326419  13  89282652          NaN 5.023752e-07
    6  p.value rs10848150  12 131057033          NaN 7.488903e-06
    7  p.value rs11011036  10  19961205          NaN 9.903649e-06

### Monte Carlo p-values

Any user-defined function of genotype and phenotype can be a test statistic.

my_custom_test.R

    my.custom.test <- function(y, x) {
      # y is phenotype vector
      # x is genotype vector
    
      N = length(x)                          # number of individuals
      stat = cor(y, x)^2 * (N - 2)           # test statistic
    
      list(n.individuals=N, stat=stat)       # return a list of named entries
    }

Estimate empirical p-values using the test statistic defined by `my.custom.test`.

    gwas.adaptive.perm(
      genotype.hdfs.path="/snps",
      phenotype.hdfs.path="/pheno.RData",
      output.hdfs.path="/results-custom",
      n.permutations=1e7,
      user.code="/my_custom_test.R",
      mytest="my.custom.test",
      statistic.name="stat"
    )

Fetch results.

    results = gwas.results.perm("/results-custom")
    subset(results, p.value<.0001)


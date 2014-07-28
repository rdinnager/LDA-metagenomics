What I want to do here is find out if Latent Dirichlet Allocation and
similar topic models are suitable for metagenomics data. I want to test
if the probability of a k-mer (or genomic 'word'), or it's frequency in
a genome can be approximated by the frequency of that k-mer in a
collection of short reads drawn from that genome. My intuition is that
it can, but the quality of the approximation will depend of the read
length and the chromosome length of the genome. This is because
overlapping regions of the reads could mess up the approximation, if
they are not distibuted uniformly throughout the genome. I believe they
should be for the most part, but this could break down near the ends of
chromosomes (e.g. reads that are too close to the ends will be tossed
because they will be too short, and so the reads just to the other side
will have less overlaps, meaning the kmers here may be
underrepresented). Since the length of genome that are a read's length
from the ends of chromosome are probably rare compared to the full
length of the genome, I suspect this won't be a big issue.

If we can approximate the k-mer frequencies of a source genome from its
reads, then we can model a mixture of reads from many genomes as a
mixture model where the frequency of a k-mer is equal to the sum of the
frequencies of that k-mer in each genome times the frequency of each
genome in that particular mixture. This is exactly what is modelled with
a Latent Dirichlet Allocation model, only it is usually used to model
documents as a mixture of topics (e.g. probability of a word in a
document is equal to probability in each topic, times the probability of
each topic being in that document). Documents are sites, topics are
genomes, and words are k-mers.

First I am going to simulate a set of genomes using the software ALF
(through my package `Ralf`):

    library(magrittr)
    devtools::install_github("rdinnager/Ralf")
    library("Ralf")
    ## generate 50 genomes of 50 genes each
    genome_sim <- run_ALF("genome_sim", 50, 50, 400, 50, 0.06, 0.025, 0.0001, "temp", 
                          "/home/din02g/ALF_standalone/bin", TRUE) %>% load_ALF

Let's take a look at the simulated tree used to simulate those genomes,
to see if everything looks okay.

    plot(genome_sim$tree, cex = 0.65, no.margin = TRUE)

![plot of chunk
unnamed-chunk-2](./assumption_test_files/figure-markdown_strict/unnamed-chunk-2.svg)

The next step it to concatenate the 50 independently simulated genes
into one long genomic sequence. We will assume this constitutes a single
genome unit, such as a chromosome.

    genome_chrom <- ALF_cat(genome_sim)
    ## convert dna to DNAStringSet for analysis with Biostrings
    genome_dna <- DNAStringSet(sapply(genome_chrom$dna, function(x) unlist(x)))
    genome_dna

    ##   A DNAStringSet instance of length 50
    ##      width seq                                         names               
    ##  [1] 67500 GAGGAGACAAGACACAAGAC...TTCCGCGATATACTGCGCAC SE001
    ##  [2] 67200 GAGCCTACAAGCCACAAGAA...TTCTGCGATATACTGCGCAC SE002
    ##  [3] 67701 GAGGCTTCACAACATAAGAC...TTCTGCGATATACTGCGCAC SE003
    ##  [4] 67281 GAGGCAACCAATCACAAGAA...TTCTGCGATATACTGCGCAC SE004
    ##  [5] 67311 GAGTATACATCTCACAAGAA...TTCTGCGATATACTGCGCAC SE005
    ##  ...   ... ...
    ## [46] 67461 GAGCCAACAAGCCACAAGAC...TTCTGCGATATACTGCGCAC SE046
    ## [47] 67200 GAGGAGACACGACACAAGAC...TTCTGCGATATACTGCGCAC SE047
    ## [48] 66723 GAGGAGACAAGGCACAAGGC...TTCTGCGATATACTGCGCAC SE048
    ## [49] 67443 GATCCAACAAGCCACAAGAC...TTCTGCGATATACTGCGCAC SE049
    ## [50] 67446 GATCCAACAAGCCACAAGAC...TTCTGCGATATACTGCGCAC SE050
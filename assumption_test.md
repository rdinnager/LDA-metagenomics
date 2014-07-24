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

First I am going to simulate a set of genome using the software ALF:

    #source("/home/din02g/Google Drive/AFD-Project/R/ALF.R")
    print("Testing!")

    ## [1] "Testing!"
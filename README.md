<p align="center"><img src="logo.png" alt="Kleborate" width="400"></p>

Kleborate is a tool to screen _Klebsiella_ genome assemblies for:
 * MLST sequence type
 * species (e.g _K. pneumoniae_, _K. quasipneumoniae_, _K. variicola_, etc.)
 * virulence genes: _ybt_, _clb_, _iro_ and _iuc_ loci, and the _rmpA_ and _rmpA2_ hypermucoidy genes
 * antimicrobial resistance genes, including quinolone resistance SNPs and colisitin resistance gene truncations
 * K and O capsule types, via the tool [Kaptive](https://github.com/katholt/Kaptive)



## Background

_Klebsiella pneumoniae_ (_Kp_) is a commensal bacterium that causes opportunistic infections, with a handful of hypervirulent lineages recognised as true human pathogens. Evidence is now mounting that other _Kp_ strains carrying acquired siderophores (yersiniabactin, salmochelin and aerobactin) and/or the genotoxin colibactin are also highly pathogenic and can cause invasive disease. 

We recently explored the diversity of the _Kp_ integrative conjugative element (ICEKp), which mobilises the yersiniabactin locus _ybt_, using genomic analysis of a diverse set of 2499 _Kp_ (see [this preprint](http://biorxiv.org/content/early/2017/01/04/098178)). Overall, we found _ybt_ in about a third of all _Kp_ genomes and _clb_ in about 5%.

We identified 17 distinct lineages of _ybt_ embedded within 14 structural variants of ICEKp (some of which include the colibactin _clb_ or salmochelin _iro_ synthesis loci, annotated reference sequences for each ICEKp variant are included in the [data directory](https://github.com/katholt/Kleborate/tree/master/kleborate/data) of this repository) that can integrate at any of four tRNA-Asn sites in the chromosome. Our analyses reveal hundreds of ICEKp transmission events affecting hundreds of chromosomal _Kp_ lineages, including nearly two dozen transfers into the globally disseminated carbapenem-resistant clonal group 258. Additionally, we identify a lineage of _ybt_ that is plasmid-encoded, representing a new mechanism for _ybt_ dispersal in _Kp_ populations.

Based on this analysis, we developed a MLST-style approach for identifying _ybt_ and _clb_ variants from genome data. 

Our goal is to help identify emerging pathogenic _Kp_ lineages, and to make it easy for people who are using genomic surveillance to monitor for antibiotic resistance to also look out for the convergence of antibiotic resistance and virulence.

To help facilitate that, in this repo we share the new _ybt_ and _clb_ schemes ([data](https://github.com/katholt/Kleborate/tree/master/kleborate/data)), annotated ICEKp structures ([ICEKp_references](https://github.com/katholt/Kleborate/tree/master/kleborate/ICEKp_references)) and code for genotyping virulence and resistance genes in _K. pneumoniae_. A table of pre-computed results for 2500 public Klebs genomes is also provided in the [data directory](https://github.com/katholt/Kleborate/tree/master/data).

If you are interested in inferring capsule types from genome data, see the [Kaptive](https://github.com/katholt/Kaptive) repo.

#### Info and Contacts

Kleborate is in active development so please post bugs and feature requests to the GitHub [issues tracker](https://github.com/katholt/Kleborate/issues).

If you use it, please cite the preprint: Lam et al, 2017 [https://doi.org/10.1101/098178](http://biorxiv.org/content/early/2017/01/04/098178)

For more on our lab, including other software, see [http://holtlab.net](http://holtlab.net)


## Let's get genotyping!

Just want to get cracking with screening a bunch of _K. pneumoniae_ genome assemblies? Use the `kleborate` command. This will detect the MLST sequence type of the strain, genotype the _ybt_ and _clb_ loci, determine the _wzi_ (capsule synthesis gene) allele and also check for presence/absence of the acquired siderophores salmochelin (_iro_) and aerobactin (_iuc_) loci and the hypermucoidy genes _rmpA_ and _rmpA2_ (allelic typing of these should be available soon)

For convenience, we provide code for screening for acquired resistance genes, quinolone-resistance determining mutations in _gyrA_ and _parC_, and colistin-resistance caused by truncation of the _mgrB_ and _pmrB_ genes. These screens are included in `kleborate` when you use the `--resistance` option, or you can use the standalone script [`resBLAST.py`](kleborate/resBLAST.py).

(If you haven't got good assemblies yet, try our [Unicycler](https://github.com/rrwick/Unicycler) assembler which works great on Illumina or hybrid Illumina + Nanopore/PacBio reads)


#### Requirements

* Python 2.7
* [setuptools](https://pypi.python.org/pypi/setuptools) (required to install Kleborate)
  * To install: `pip install setuptools`
* BLAST+ command line tools (`makeblastdb`, `blastn`, etc)
  * Version 2.2.30 or later is needed, as earlier versions have a bug with the culling_limit parameter.
* [Mash](https://github.com/marbl/Mash) is required to use the `--species` option


#### Installation

```bash
git clone --recursive https://github.com/katholt/Kleborate.git
cd Kleborate
python setup.py install
kleborate -h
```


#### Run without installation

You don't have to install Kleborate to use it. You can instead use the `kleborate-runner.py` script:

```bash
git clone --recursive https://github.com/katholt/Kleborate.git
Kleborate/kleborate-runner.py -h
```


#### Basic usage

```
# Screen some genomes for MLST and virulence loci:
kleborate -o detailed_results.txt -a *.fasta

# Also screen for resistance genes:
kleborate --resistance -o detailed_results.txt -a *.fasta

# Turn on all of Kleborate's optional screens (resistance gene, species check and Kaptive
# for both K and O loci):
kleborate --all -o detailed_results.txt -a *.fasta
```

See below for more details, examples and outputs.



## About the MLST schemes

We have created two separate schemes: one for yersiniabactin sequence types (YbST) and one for colibactin sequence types (CbST). See [this preprint](http://biorxiv.org/content/early/2017/01/04/098178) for full details.

MLST-style schemes are included in the [_Klebsiella pneumoniae_ BIGSdb hosted at the Pasteur Institute](http://bigsdb.pasteur.fr/klebsiella/klebsiella.html), and are also included in the [data directory](https://github.com/katholt/Kleborate/tree/master/kleborate/data) of this repository. The schemes include all known alleles for the genes that make up the yersiniabactin and colibactin synthesis loci that are mobilised by the _Klebsiella_ ICEKp.

#### YbST - Yersiniabactin Sequence Types

YbST sequences cluster into 17 distinct lineages of _ybt_, which are each associated with a particular ICEKp structure (see the paper for details). Three lineages (_ybt_ 1, _ybt_ 12, _ybt_ 17) are associated with the ICEKp10 structure in which the _clb_ colibactin locus is found; _ybt_ 4 is plasmid-encoded; and all other lineages correspond to a single ICEKp.

#### CbST - Colibactin Sequence Types

CbST sequences cluster into 3 lineages, which are each associated with a single _ybt_ lineage (_clb_ 1, _ybt_ 12; _clb_ 2, _ybt_ 1; _clb_ 3, _ybt_ 17) and the ICEKp10 structure.



## Genotypes of publicly available strains

A table of pre-computed yersiniabactin, colibactin,  capsule locus and chromosomal MLST assignments for 2500 public Klebs genomes is provided in the [data directory](https://github.com/katholt/Kleborate/tree/master/data).



## Detailed Usage - Typing genome assemblies using Kleborate

Kleborate can be used to determine chromosomal, yersiniabactin and colibactin genotypes from assembled draft or complete genomes. It also reports presence/absence of acquired siderophores salmochelin (_iro_) and aerobactin (_iuc_) loci and the hypermucoidy genes _rmpA_ and _rmpA2_ (allelic typing of these should be available soon, currently we just screen for the alleles in the virulence plasmid pLVPK). We also extract the _wzi_ gene allele to give an idea of the capsule type, but these are not totally predictive so we suggest you use our dedicated capsule typing tool [Kaptive](https://github.com/katholt/Kaptive) for this. For convenience, Kleborate can optionally screen for acquired resistance genes as well (using the SRST2-formatted version of the ARG-Annot database, with the core Klebs genes _oqxA_ and _oxqB_ removed).

A summary of sequence types and ICE/lineage information is printed to standard out; full allele calls are saved to a file specified by `-o`. See below for details of output formats.


#### Usage:

```
usage: kleborate -a ASSEMBLIES [ASSEMBLIES ...] [-r] [-s] [--kaptive_k]
                 [--kaptive_o] [-k] [--all] [-o OUTFILE]
                 [--kaptive_k_outfile KAPTIVE_K_OUTFILE]
                 [--kaptive_o_outfile KAPTIVE_O_OUTFILE] [-h] [--version]

Kleborate: a tool for characterising virulence and resistance in Klebsiella

Required arguments:
  -a ASSEMBLIES [ASSEMBLIES ...], --assemblies ASSEMBLIES [ASSEMBLIES ...]
                        FASTA file(s) for assemblies

Screening options:
  -r, --resistance      Turn on resistance genes screening (default: no
                        resistance gene screening)
  -s, --species         Turn on Klebsiella species identification (requires
                        Mash, default: no species identification)
  --kaptive_k           Turn on Kaptive screening of K loci (default: do not
                        run Kaptive for K loci)
  --kaptive_o           Turn on Kaptive screening of O loci (default: do not
                        run Kaptive for O loci)
  -k, --kaptive         Equivalent to --kaptive_k --kaptive_o
  --all                 Equivalent to --resistance --species --kaptive

Output options:
  -o OUTFILE, --outfile OUTFILE
                        File for detailed output (default:
                        Kleborate_results.txt)
  --kaptive_k_outfile KAPTIVE_K_OUTFILE
                        File for full Kaptive K locus output (default: do not
                        save Kaptive K locus results to separate file)
  --kaptive_o_outfile KAPTIVE_O_OUTFILE
                        File for full Kaptive O locus output (default: do not
                        save Kaptive O locus results to separate file)

Help:
  -h, --help            Show this help message and exit
  --version             show program's version number and exit
```


#### Test on well known genomes:

```bash
## GET CODE
git clone https://github.com/katholt/Kleborate 

## GET DATA FROM NCBI

# NTUH-K2044 (ST23, ybt 2; ICEKp1)
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/009/885/GCA_000009885.1_ASM988v1/GCA_000009885.1_ASM988v1_genomic.fna.gz
gunzip GCA_000009885.1_ASM988v1_genomic.fna.gz
mv GCA_000009885.1_ASM988v1_genomic.fna NTUH-K2044.fna

# Kp1084 (ST23, ybt 1; ICEKp10, clb 2)
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/294/365/GCA_000294365.1_ASM29436v1/GCA_000294365.1_ASM29436v1_genomic.fna.gz
gunzip GCA_000294365.1_ASM29436v1_genomic.fna.gz
mv GCA_000294365.1_ASM29436v1_genomic.fna Klebs_Kp1084.fna

# HS11286 (ST11, ybt 9; ICEKp3)
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/240/185/GCA_000240185.2_ASM24018v2/GCA_000240185.2_ASM24018v2_genomic.fna.gz
gunzip GCA_000240185.2_ASM24018v2_genomic.fna.gz
mv GCA_000240185.2_ASM24018v2_genomic.fna Klebs_HS11286.fna

# MGH 78578 (ST38, no ICE)
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/016/305/GCA_000016305.1_ASM1630v1/GCA_000016305.1_ASM1630v1_genomic.fna.gz
gunzip GCA_000016305.1_ASM1630v1_genomic.fna.gz
mv GCA_000016305.1_ASM1630v1_genomic.fna MGH78578.fna

# RUN KLEBORATE
./Kleborate/kleborate-runner.py -o results.txt -a *.fna

## EXPECTED OUTPUT
strain  ST  virulence_score Yersiniabactin  YbST  Colibactin  CbST  aerobactin  salmochelin hypermucoidy  wzi KL
Klebs_HS11286 ST11  1 ybt 9; ICEKp3 15  - 0 - - - wzi74 KL103
Klebs_Kp1084  ST23  3 ybt 1; ICEKp10  47  clb 2 37  - - - wzi172  KL1
MGH78578  ST38  0 - 0 - 0 - - - wzi50 KL15 (KL17,KL51,KL52)
NTUH-K2044  ST23  3 ybt 2; ICEKp1 326 - 0 iucABCD iroBCDN;iroBCDN rmpA;rmpA wzi1  KL1

# NOTE: NTUH-K2044 has two copies each of the iro and rmpA loci (one on the virulence plasmid with iuc, and one in ICEKp1).

# RUN KLEBORATE with resistance gene screening turned on:
./Kleborate/kleborate-runner.py --resistance -o results_with_resistance.txt -a *.fna

```



## Output

A tabulated summary is printed to standard out; details of the MLST analysis are printed to the file specified by -o.

#### Klebs, yersiniabactin and colibactin MLST
* Kleborate makes an effort to report the closest matching ST / clonal group if a precise match is not found.
* Imprecise allele matches are indicated with a \*
* Imprecise ST calls are indicated with -nLV, where n indicates the number of loci that disagree with the ST reported. So 258-1LV indicates a single-locus variant of (SLV) of ST258, i.e. 6/7 loci match the ST258 alleles.
* For _ybt_ and _clb_ schemes, the associated lineage and/or ICEKp structure are reported along with the ST. If these loci are not detected, the ST reported is 0.

#### Virulence locus detection
* Results in these columns should be interpreted as simply binary yes/no calls.
* By default, a gene is called as present if it is detected in a single sequence with >90% identity and >80% coverage of the allele sequence from the virulence plasmid pLVPK. Note that rmpA and rmpA2 are ~85% homologous and are reported separately.
* If multiple hits to the same query sequence are found in a given assembly, we attempt to report these separately. The NTUH-K2044 genome carries iroBCDN and rmpA in two locations (one on the virulence plasmid, one in the ICEKp1), and this should be reported as iroBDCN;iroBCDN and rmpA;rmpA as seen in the example above.

#### Virulence score
This is a simple score from 0-3 to roughly quantify how virulent the strain is:
* 0 = no virulence loci
* 1 = only yersiniabactin
* 2 = one or more virulence loci other than yersiniabactin (colibactin, aerobactin, salmochelin, hypermucoidy or any combination of these)
* 3 = yersiniabactin and one or more other virulence loci

#### Wzi gene allele (marker of capsule type)
* The closest match amongst the _wzi_ alleles in the BIGSdb will be reported.
* This is a marker of capsule locus (KL) type, which is highly predictive of capsule (K) serotype. Although there is not a 1-1 relationship between wzi allele and KL/K type, it is quite predictive (see [Wyres et al, MGen 2016](http://mgen.microbiologyresearch.org/content/journal/mgen/10.1099/mgen.0.000102)).
* This can a handy way of spotting the virulence-associated types (wzi=K1, wzi2=K2, wzi5=K5); or spotting capsule switching within clones, e.g. you can tell which ST258 lineage you have from the wzi type (wzi154: the main lineage II; wzi29: recombinant lineage I; others: probably other recombinant lineages)
* Note for optimal capsule type prediction you should use our dedicated capsule typing tool [Kaptive](https://github.com/katholt/Kaptive).

#### Resistance gene detection
* Here we are screening against the ARG-Annot database of acquired resistance genes ([SRST2](https://github.com/katholt/srst2) version), which includes allelic variants.
* Kleborate attempts to report the best matching variant for each locus in the genome
* Imprecise allele matches are indicated with \*
* If the length of match is less than the length of the reported allele (ie a partial match), this is indicated with ?
* Note that the beta-lactamases ampH and SHV (narrow spectrum) are core genes in _K. pneumoniae_ so should be detected in most genomes
* If you see LEN or OKP beta-lactamases rather than SHV, you probably have _K. variicola_ (LEN) or _K. quasipneumoniae_ (OKP) rather than _K. pneumoniae_ (see [this paper](http://www.pnas.org/content/112/27/E3574.long) for clarification)
* Note that _oqxAB_ are also core genes  in _K. pneumoniae_, but have been removed from this version of the ARG-Annot DB as they don't actually confer resistance to fluoroquinolones
* In addition to acquired genes, we also check for the known quinolone resistance SNPs (GyrA 83 & 87; ParC 80 & 84)
* Results are grouped by drug class (according to the [ARG-Annot](https://www.ncbi.nlm.nih.gov/pubmed/24145532) DB), with beta-lactamases broken down into Lahey classes, as follows: 
  * AGly (aminoglycosides)
  * Bla (beta-lactamases)
  * Bla_broad (broad spectrum beta-lactamases)
  * Bla_broad_inhR (broad spectrum beta-lactamases with resistance to beta-lactamase inhibitors)
  * Bla_Carb (carbapenemase)
  * Bla_ESBL (extended spectrum beta-lactamases)
  * Bla_ESBL_inhR (extended spectrum beta-lactamases with resistance to beta-lactamase inhibitors)
  * Fcyn (fosfomycin)
  * Flq (fluoroquinolones)
  * Gly (glycopeptides)
  * MLS (macrolides)
  * Phe (phenicols)
  * Rif (rifampin)
  * Sul (sulfonamides)
  * Tet (tetracyclines)
  * Tmt (trimethoprim)


#### Resistance score
This is a simple score from 0-2 to roughly quantify how resistant the strain is:
* 0 = no ESBL, no carbepenemase
* 1 = ESBL, no carbepenemase
* 2 = Carbepenemase (whether or not ESBL is present)


## _Klebsiella_ species

By using the `--species` option, Kleborate will attempt to identify the species of _Klebsiella_. It does this by comparing the assembly using Mash to a curated set of _Klebsiella_ assemblies [from NCBI](https://www.ncbi.nlm.nih.gov/assembly) and reporting the species of the closest match. Kleborate considers a Mash distance of ≤ 0.01 to be a strong species match. A distance of > 0.01 and ≤ 0.03 is a weak match and might indicate that your sample is a novel lineage or a hybrid between multiple _Klebsiella_ species. 

Here is an annotated tree of the reference assemblies, made by [mashtree](https://github.com/lskatz/mashtree):
<p align="center"><img src="images/species_tree.png" alt="Klebsiella species tree" width="90%"></p>

Kleborate is designed for the well studied group of species at the top right of the tree which includes the "big three": _pneumoniae_, _quasipneumoniae_ (two subspecies) and _variicola_. _K. quasivariicola_ is more recently characterised and described here: [Long 2017](http://genomea.asm.org/content/5/42/e01057-17). The Kp5 group does not yet have a species name and was described in this paper: [Blin 2017](http://onlinelibrary.wiley.com/doi/10.1111/1462-2920.13689/abstract). More distant _Klebsiella_ species (_oxytoca_, _michiganensis_ and _aerogenes_) are less well characterised and isolates are often labelled inconsistently. We have defined these species using the names most commonly assigned to the clades, but these species delineations deserve closer attention.

Kleborate will also call other species in Enterobacteriaceae, as different species sometimes end up in _Klebsiella_ collections. These names are again assigned based on the clades in a mashtree, but were not as carefully curated as the _Klebsiella_ species (so take them with a grain of salt).



## Typing direct from Illumina reads

MLST assignment can also be achieved direct from reads using [SRST2](https://github.com/katholt/srst2). Steps are 

* download the YbST and CbST allele sequences and profile tables from the [data directory](https://github.com/katholt/Kleborate/tree/master/kleborate/data) in _this_ repository
* Install [SRST2](https://github.com/katholt/srst2) if you don't already have it (`git clone https://github.com/katholt/srst2`); 
* Run SRST2, setting the `--mlst_scheme` and `--mlst_definitions` to point to the YbST or CbST allele sequences and profile tables like this:

```
srst2 --input_pe reads_1.fastq.gz reads_2.fastq.gz --output YbST --log --mlst_db ybt_alleles.fasta --mlst_definitions YbST_profiles.txt

srst2 --input_pe reads_1.fastq.gz reads_2.fastq.gz --output CbST --log --mlst_db colibactin_alleles.fasta --mlst_definitions CbST_profiles.txt
```

Note that currently you can only run SRST2 with one MLST scheme at a time, so in order to type MLST, YbST and CbST you will need to run three separate commands:

```
srst2 --input_pe reads_1.fastq.gz reads_2.fastq.gz --output YbST --log --mlst_db ybt_alleles.fasta --mlst_definitions YbST_profiles.txt

srst2 --input_pe reads_1.fastq.gz reads_2.fastq.gz --output CbST --log --mlst_db colibactin_alleles.fasta --mlst_definitions CbST_profiles.txt

srst2 --input_pe reads_1.fastq.gz reads_2.fastq.gz --output Klebs --log --mlst_db Klebsiella_pneumoniae.fasta --mlst_definitions kpnuemoniae.txt
```




## Kleboration

Kleborate is under active development with many other Klebs genomic analysis tools and projects in progress. 

Please get in touch via the issues tracker if you have any issues, questions or ideas.

-------------

Stop! Kleborate and listen

ICEKp is back with with my brand-new invention

If there was a problem, Klebs'll solve it

Check out the hook while Klebs evolves it

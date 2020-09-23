==========
AlphaPhase
==========

.. .. contents::
..    :depth: 5

Introduction
============
|ap| is a software package for phasing genotype data. The program implements methods determine phase using an extended Long Range Phasing (Kong et al., 2008, Hickey et al., 2010) and Haplotype Library Imputation (Hickey et al., 2009 and 2010). |ap| consists of a single program. All information on the model of analysis, input files and their layout, is specified in a parameter file.

Please report bugs or suggestions on how the program / user interface / manual could be improved or made more user friendly to `alphagenes@roslin.ed.ac.uk <alphagenes@roslin.ed.ac.uk>`_

Program History
---------------

The first version of AlphaPhase was written by John Hickey and implemented Long Range Phasing and Haplotype Library Imputation.  In 2016 Daniel Money implemented new functionality to enable the phasing of larger datasets.  This new functionality allowed Long Range Phasing to be performed on subsets of animals rather than on all animals.

Availability
------------

|ap| is available from the AlphaGenes website, `http://www.alphagenes.roslin.ed.ac.uk <http://www.alphagenes.roslin.ed.ac.uk/>`_.

Material available comprises the compiled programs for 64 bit Windows, Linux, and Mac OSX machines, together with this document.

Conditions of use
-----------------

|ap| is part of a suite of software that our group has developed. It is fully and freely available for all use under the MIT License.

Suggested Citation:

  Hickey et al. (2011) A combined long range phasing and long haplotype imputation method to phase SNP genotypes. Genetics Selection Evolution.

Disclaimer
----------

While every effort has been made to ensure that |ap| does what it claims to do, there is absolutely no guarantee that the results provided are correct. Use of |ap| is entirely at your own risk.

Advertisement
-------------

Your welcome to check out our Gibbs sampler (`AlphaBayes <http://www.alphagenes.roslin.ed.ac.uk/software-packages/alphabayes/>`_) specifically designed for GWAS and Genomic Selection.

Descriptor of method
--------------------

The method implemented in |ap| is described in detail in Hickey et al. (2010).

Using AlphaPhase
================

A number of changes have been made internally to increase the speed and accuracy of |ap| compared to previous versions.

A change has been made to the ``AlphaPhaseSpec.txt`` file. This allows the program to be run in an ``Offset`` mode or a ``NotOffset`` mode. The ``NotOffset`` mode is as before, the ``Offset`` mode is designed to create overlaps between cores. First running the program in ``NotOffset`` phases several cores, then running the program in ``Offset`` mode moves the start of the cores to halfway along the first core, thereby creating 50% overlaps between cores for the ``NotOffset`` mode and the ``Offset`` mode. The mode option is specified at the end of the ``GeneralCoreLength`` line.

AlphaPhase has been modified to more efficiently deal with large datasets.  In short, AlphaPhase can now run Long Range Phasing on a subset of animals rather than all animals.  This option is controlled by ``IterateMethod``, ``IterateSubsetSize`` and ``IterateIterations``.

An option has now been added to AlphaPhase that means that, during the Haplotype Library Imputation stage, a haplotype is only used for imputation if it has already been discovered the given number of times.  This is controlled by ``MinHapFreq``.

Additional new options (described below) are ``CoreAtTime`` and ``Cores`` which add additionally options which may be beneficial when using large datasets.  An option, ``Library`` has also been added that allows a pre-existing haplotype library to be used to seed the imputation library.

Input files
-----------
Two files are required when running the program. A third (Pedigree file) can also be supplied. A fourth (the true phase) can also be supplied if simulated data is being used and performance is required to be tested.

The two primary input files are ``AlphaPhaseSpec.txt`` (a parameter file used to control the program) and a genotype file.


AlphaPhaseSpec.txt
^^^^^^^^^^^^^^^^^^

An example of ``AlphaPhaseSpec.txt`` is shown in Figure 1. Everything to the left of the comma should not be changed. The program is controlled by changing the input top the right of the comma.  All options up to ``TruePhaseFile``.  Options occurring after this option are optional although if an option is specified all previous options must also be present.::

  PedigreeFile                      ,PreSpecifiedPedigree.txt
  GenotypeFile                      ,Genotype.txt,GenotypeFormat
  NumberOfSnp                       ,2000
  GeneralCoreAndTailLength          ,300
  GeneralCoreLength                 ,100,``NotOffset``
  UseThisNumberOfSurrogates         ,10
  PercentageSurrDisagree            ,10.00
  PercentageGenoHaploDisagree       ,0.00
  GenotypeMissingErrorPercentage    ,0.00
  NrmThresh                         ,0.00
  FullOutput                        ,1
  Graphics                          ,0
  Simulation                        ,1
  TruePhaseFile                     ,TruePhase.txt
  CoreAtTime                        ,0
  IterateMethod                     ,Off
  IterateSubsetSize                 ,500
  IterateIterations                 ,1
  Cores                             ,1,Combine
  MinHapFreq                        ,1
  Library                           ,"PreviousRun/HaplotypeLibrary"

Below is a description of what each line is for. It is important to note that ``AlphaPhaseSpec.txt`` is case sensitive.

``PedigreeFile`` gives the name of the file containing the pedigree information. If there is no pedigree file available include ``NoPedigree`` in place of a pedigree file name.

``GenotypeFile`` gives the name of the file containing the genotypes, followed by a comma, followed by the format of the genotype file. There are three possible formats, ``GenotypeFormat`` (where the genotypes are coded as ``0``, ``1``, and ``2``) and ``UnorderedFormat`` (where the genotypes as unordered alleles coded as ``1``, and ``2``). Further details are given in the Genotype File format description given below.

``NumberOfSnp`` gives the number of SNP in the genotype file.

``GeneralCoreAndTailLength`` gives the overall length in terms of numbers of SNPs of the core and its adjacent tails. For example if the GeneralCoreLength (described below) is 100 and the GeneralCoreAndTailLength is 300 this means that the core is 100 SNPs long and the tails are the 100 SNPs adjacent to each end of the core, thus the length of the core and tail is 300 SNPs long. At the end of a chromosome, the tail can only extend in one direction. Thus in this case the core and tail length would be only be 200 SNPs, the 100 SNPs in the core, and the 100 SNPs adjacent to the core.

``GeneralCoreLength`` gives the overall length in terms of numbers of SNPs of the core. The GeneralCoreLength can never be longer than the GeneralCoreAndTailLength. The mode is also set at the end of this line. The two options are “``Offset``” and “``NotOffset``”.

``UseThisNumberOfSurrogates`` give the number of surrogates across which information pertaining to phase must be accumulated before phase can be declared.

``PercentageSurrDisagree`` gives the percentage of surrogates that are allowed to conflict with the majority of the surrogates and still have phased declared. For example a 10.00 (10%) value means that if information about phase is accumulated across 10 surrogates and 9 of them indicate phase is in one direction and 1 indicates it is in the other, phase is declared to be in the direction of the 9. But if these counts are 8 in one direction and 2 in the other, phase is undeclared (i.e. the minority is more than 10%).

``PercentageGenoHaploDisagree`` gives the percentage of disagreement across all SNPs in a core which are allowed to disagree between the genotype and the genotype suggested by sum of the alleles in the candidate pair of haplotypes for the candidate haplotypes to be still considered to be valid. For example a 1.00 (1%) value means that across a core of 100 SNPs 1 SNP is allowed to conflict between its actual genotype and the genotype comprised of the sum of the alleles of the candidate haplotypes.

``GenotypeMissingErrorPercentage`` gives the percentage of SNPs that are allowed to be missing or in conflict across the entire core and tail length during surrogate definition. A 1.00 (1%) value means that across a GeneralCoreAndTailLength of 300 SNPs, 3 of these SNPs are allowed to be missing or in disagreement between two otherwise compatible surrogate parents. Thus these two individuals are allowed to be surrogate parents of each other in spite of the fact that 1% of their genotypes are missing or are in conflict (i.e. opposing homozygotes).

``NrmThresh`` gives the maximum value (between 0.00 and 1.00) that the coefficient of relationship can take between a dummy sire and the true dam when pedigree information is used to partition surrogates in situations where parents are not genotyped. Section 2b (iv.) of Appendix A of Hickey et al. (2010) gives more details.

``FullOutput`` determines whether the extra output files are suppressed or not. A value of ``1`` gives the full output. A value of ``0`` suppresses the full output.

``Graphics`` determines whether the graphical output is invoked or not. The graphical components are not yet functional hence a value of ``0`` is required here.

``Simulation`` determines whether the analysis involves simulated data where the true phase is known and performance measurement is required or not. A value of ``1`` gives indicates that it is a simulation. A value of ``0`` indicates that it is not a simulation.

``TruePhaseFile`` gives the name of the file containing the true phase when working with simulation. The program does not read this line when the value of the line above is set to ``0`` hence it is irrelevant when working with real data.

``CoreAtTime`` determines whether AlphaPhase stores only a single core in memory at any one time. A value of ``1` means that Alphaphase only keeps one core in memory which means AlphaPhase uses less memory but is slower.  A value of ``0`` (default) means that AlphaPhase will keep all cores in memory.

``IterateMethod`` determines what method AlphaPhase uses to determine the animals to be used in each Long Range Phasing subset.  ``Off`` (default) means all animals are included in a single subset.  ``RandomOrder`` means animals are assigned to subsets randomly while ``InputOrder`` means animals are assigned to subsets in the order they appear in the genotype file.

``IterateSubsetSize`` determines the number of animals to be included in each subset when ItterateMethod is not Off.  Default is 200.

``IterateIterations`` determines the maximum number of times each animal will be included in a subset.  Default is 1.

``Cores`` consists of two parameters indicating the cores to be calculated.  The first parameter indicates the core to start at while the second parameter indicates the cores to finish at.  Both parameters can be either the number of a core or ``Combine``.  ``Combine`` means perform the final combining step to combine all the cores into a single result.  For example ``1, Combine`` (default) means calculate each core and then perform the combining step.

``MinHapFreq`` gives the minimum number of times a haplotype will need to have been discovered before it is imputed from in the Haplotype Library Imputation step.  Default is 1.

``Library`` gives the location of a preexisting library for use as a seed library for the Haplotype Library Imputation stage.  A value of ``None`` (default) means no pre-existing library is used.

Advice on values for parameters
"""""""""""""""""""""""""""""""

``GeneralCoreLength`` and ``GeneralCoreAndTailLength`` Short cores and intermediate core and tail lengths give the best results. However the algorithm is robust to small variations about what the optimal is likely to be. For 60,000 SNP density a core length of 100 SNPs and a core and tail length of 300 to 500 SNP is advisable. For 300,000 SNP density a core length of 400 SNPs and a core and tail length of 1200 to 2000 SNP is advisable.

``UseThisNumberOfSurrogates`` and ``PercentageSurrDisagree`` Good results were obtained using values of 10 for UseThisNumberOfSurrogates and 10.00 (10%) for PercentageSurrDisagree.

``PercentageGenoHaploDisagree`` and ``GenotypeMissingErrorPercentage`` It is best to be stringent with the editing of data (i.e. remove animals with large numbers of missing or poorly called SNP and remove SNPs with large numbers of or poorly called missing individuals) and then use low values for these parameters (e.g. 0.00 (0%) or 1.00 (1%)).

It is advisable to play with all of these parameters to fine tune them for a particular data set. Making ``GeneralCoreAndTailLength`` too short and ``GenotypeMissingErrorPercentage`` too high can increase the computational time considerably and can give poorer phasing performance. The trends in Hickey et al. (2010) can be used to give a feel for what is sensible.

Data format
^^^^^^^^^^^

Pedigree file
"""""""""""""

The pedigree file should have three columns, individual, father, and mother. It should be space or comma separated with for missing parents coded as 0. No header line should be included in the pedigree file both numeric and alphanumeric formats are acceptable. The pedigree does not have to be sorted in any way as the program automatically does this. If no pedigree file is available ``NoPedigree`` should be given in place of a pedigree file name in ``AlphaPhaseSpec.txt``.

Genotype file
"""""""""""""

The genotype information should be contained in a single file containing 1 line for each individual. The first column of this file should contain the individual’s identifier with numeric and alphanumeric formats being acceptable. The next columns should contain the SNP information with two formats being acceptable, ``GenotypeFormat`` and ``UnorderedFormat``.

``GenotypeFormat`` has a single column for each SNP where the genotypes are coded as ``0``, ``1``, and ``2`` and missing genotypes are coded as ``3`` or ``9``, with ``0`` being homozygous ``aa``, ``1`` being heterozygous ``aA`` or ``Aa``, and ``2`` being homozygous ``AA``.

``UnorderedFormat`` has two consecutive columns for each SNP, with ``aa`` being coded as ``1 1``, ``aA`` and ``Aa`` being coded as ``1 2`` or ``2 1`` and ``AA`` being coded as ``2 2``. Missing genotypes can take any other numeric format (e.g. ``3 3``) Examples of these formats are included in the examples subdirectory. The genotype file should not have a header line.

Output
------
The output of |ap| is organised into a number of sub directories (``PhasingResults``, and in the case of when simulated data is used Simulation). A description of what is contained within these folders is given below.

PhasingResults
^^^^^^^^^^^^^^

``PhasingResults`` contains the primary results file and an index file with its coordinates. ``FinalPhase.txt`` contains the final phased output for each individual. It has two rows for each individual and a column for each locus. The first column contains the individual’s identification, followed by the phased information for the SNPs in the same order as the input genotype file. The coordinates of ``FinalPhase.txt`` are contained within ``CoreIndex.txt``. By the coordinates what is meant is the start point and end point of each core (i.e. where a haplotype begins and ends). Cores are unaligned. Three columns exist in ``CoreIndex.txt``. Column 1 is the core identifier, column 2 is the starting SNP of the core, and column 3 is the ending SNP of the core.

``IndivPhaseRate.txt`` contains the percentage of alleles phased in each of the cores for each individual, with columns being Id, % phased core 1, % phased core 2.... etc.

``SnpPhaseRate.txt`` contains the percentage of individuals phased for each SNP, with the columns being SNP ordered number and % of individuals phased for that SNP.

``PhasingYield.txt`` contains the average % phased across all the individuals and all the SNPs for each core. It is a handy file for checking the performance for each core.

The directory ``HaplotypeLibrary`` contains the library of haplotypes (e.g. ``HapLib1.txt`` is the library for core 1) for each core and the directory ``Extras``. In the first column of ``HapLibX.txt`` is the haplotype Id, then its frequency, then the haplotype. ``Extras`` contains files called ``HapCommonalityX.txt`` which contain matrices of relationships between the haplotypes within a core. These relationships are calculated as the count of alleles which match each pair of haplotypes divided by the total number of SNPs in a core.

Miscellaneous
^^^^^^^^^^^^^

``Miscellaneous`` contains files which summarise the data. The allele frequency for each SNP, the genomic relationship matrix is contained within ``GenotypedMarkerNRM.txt``. The pedigree derived numerator relationship matrix between the genotyped individuals is contained within ``GenotypedNRM.txt``, a pseudo version of this showing relationships which are above the NrmThresh as 1 and below it as 0 is given in ``GenotypedPseudoNRM.txt``.

``SurrogatesX.txt`` contains a matrix of how animals are surrogate of each other for core X. A ``1`` means it is a surrogate of one of the clusters (i.e. paternal / maternal) and a ``2`` means it is surrogate of the other. The labelling paternal / maternal is arbitrary. ``SurrogatesSummaryX.txt`` contains six columns. Column 1 is the Id, column 2 is the count of cluster 1 surrogates (e.g. Paternal), column 3 is the count of cluster 2 surrogates (e.g. Maternal), column 4 is the count of surrogates that are in both clusters (e.g. Paternal and Maternal), column 5 is the count of all surrogates, and column six is a code for how the surrogates were partitioned (``1`` = both parents genotyped, ``2`` = sire genotyped and used for partitioning, ``3`` = dam genotyped and used for partitioning, ``4`` = pseudo NRM partitioning, ``5`` = progeny genotyped and used for partitioning, ``6`` = k--medoid partitioning). Details on these partitioning strategies are given in Hickey et al. (2010).

``Timer.txt`` contains the time takes to complete the task.

Simulation
^^^^^^^^^^

Simulation contains files summarising the comparisons between the simulated data and the phased output. ``CoreMistakesPercent.txt`` has a row for each core, followed by an empty row followed by a row containing the average across each of the cores. The columns are % of all alleles phased correctly within a core, % of all heterozygous alleles phased correctly within a core, % of all alleles not phased, % of heterozygous alleles not phased, percentage of all alleles incorrectly phased, and percentage of heterozygous alleles incorrectly phased. In ``IndivMistakesPercentX.txt`` column 1 is the Id, column 2 is the count of cluster 1 surrogates (e.g. Paternal), column 3 is the count of cluster 2 surrogates (e.g. Maternal), and column 4 is the count of all surrogates for each individual for core X.

Column 5 and 6 are the % of all alleles correctly phased within a core for the paternal and maternal alleles. Column 7 and 8 are the % of all alleles not correctly phased within a core for the paternal and maternal alleles. Column 9 and 10 are the % of all alleles incorrectly correctly phased within a core for the paternal and maternal alleles. The next 6 columns are the same as the previous 6 except that they refer to the heterozygous SNPs. The next six columns are also the same except that they refer to the missing SNPs while the final six columns refer to the SNPs simulated to have genotype error (must be identified in the program (contact John Hickey)). ``IndivMistakesX.txt`` contains the raw counts of what ``IndivMistakesPercentX.txt`` contains as percentages. ``MistakesX.txt`` contains the raw individual by SNP mistakes, with alleles phased correctly coded as ``1``, not phased coded as ``9``, and incorrectly phased coded as ``5``.

.. Examples
.. ========

.. Phasing using pedigree information
.. ----------------------------------
.. Examples are contained in the folder ``PhasingWithPedigreeInformation``.

.. GenotypeFormat
.. ^^^^^^^^^^^^^^

.. An example using the genoptype format for the genotype file is available in the subdirectory GenotypeFormat of PhasingWithPedigreeInformation. ``PresSpecifiedPedigree.txt`` is file containing the pedigree file. ``60kGenotypeGF.txt`` contains the genotype information with the format GenotypeFormat . It has 2000 SNPs.

.. UnorderedFormat
.. ^^^^^^^^^^^^^^^

.. An example using the unordered format for the genotype file is available in the subdirectory UnorderedFormat of PhasingWithPedigreeInformation. ``PreSpecifiedPedigree.txt`` is file containing the pedigree file. ``60kGenotypeGF.txt`` contains the genotype information with the format UnorderedFormat. It has 2000 SNPs.

.. Phasing without using pedigree information
.. ------------------------------------------
.. Examples are contained in the folder ``PhasingWithPedigreeInformation``.

.. GenotypeFormat
.. ^^^^^^^^^^^^^^

.. An example using the genoptype format for the genotype file is available in the subdirectory GenotypeFormat of PhasingWithoutPedigreeInformation. ``60kGenotypeGF.txt`` contains the genotype information with the format GenotypeFormat . It has 2000 SNPs. ``NoPedigree`` is used in place of a pedigree file name to specify that no pedigree information is being used.

.. UnorderedFormat
.. ^^^^^^^^^^^^^^^

.. An example using the unordered format for the genotype file is available in the subdirectory UnorderedFormat of PhasingWithoutPedigreeInformation. ``60kGenotypeGF.txt`` contains the genotype information with the format UnorderedFormat. It has 2000 SNPs. ``NoPedigree`` is used in place of a pedigree file name to specify that no pedigree information is being used.

.. Phasing with a simulated scenario
.. ---------------------------------
.. To measure performance simulated data can be used where a file of the true phase is included. An example of this is given in SimulatedScenario. The true phase is contained in ``60kPhase.txt`` where there are two lines for each individual (i.e. a line for each gamete). The first column in this file contains the Id, the next columns are a column for each SNP.

.. Background reading
.. ==================

.. [1] Long range phasing and haplotype imputation for improved genomic selection calibrations. 2009. Hickey, J.M., B. P. Kinghorn and J.H.J. van der Werf. Statistical Genetics of Livestock for thePost-­‐Genomic Era. University of Wisconsin -­‐ Madison, USA May 4-­‐6, 2009

.. [2] Phasing of SNP data by combined recursive long range phasing and long range haplotype imputation. 2009. Hickey, J.M., Kinghorn, B.P., Tier, B., and van der Werf, J.H.J. Proceedings of AAABG. Pages 72 – 75.

.. [3] A recursive algorithm for long range phasing of SNP genotypes. 2009. Kinghorn, B.P., Hickey, J.M., and van der Werf, J.H.J. Proceedings of AAABG. Pages 76 – 79.

.. [4] Recursive Long Range Phasing And Long Haplotype Library Imputation: Application to Building A Global Haplotype Library for Holstein cattle. 2010. Hickey, J.M., Kinghorn, B.P., Cleveland, M., Tier, B. and van der Werf, J.H.J. (Accepted at 9th WCGALP).

.. [5] Reciprocal recurrent genomic selection (RRGS) for total genetic merit in crossbred individuals. 2010. Kinghorn, B.P., Hickey, J.M., and van der Werf, J.H.J. (Accepted at 9th WCGALP).

.. [6] Determining phase of genotype data by combined recursive long range phasing and long range haplotype imputation. Hickey, J.M., Kinghorn, B.P., Tier, B., and van der Werf, J.H.J.

.. |ap| replace:: **AlphaPhase1.2**

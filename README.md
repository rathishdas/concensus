Generating consensus sequence from the subread is one of the most prominent research field in
computational biology. The most intuitive way to approach this problem is to perform MSA (Multiple sequence
Alignment). There are several techniques of doing MSA. In this paper, we will represent two ways of generating
consensus from the pacbio data.

A. Consensus generation using Phylogeny
B. Consensus generation using CCS

A. Consensus generation using Phylogeny
-------------------------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------
./poa -read_fasta ../7864612.fa -clustal ../7864612_clustal.aln blosum80.mat -tolower -v
Data Preparation:
----------------------------------
Using samtools read the sequences of desired id number and save them

   samtools view -h m54113_160913_184949.subreads_sorted.bam | head -20000 | grep 7864612 | awk '{print $10}' > 7864612_tmp.fa
      
	Here
		“M54113_160913_184949.subreads_sorted.bam” is ID sorted input ‘.bam’ file
		
		“7864612” is desired id number
		
		“7864612_tmp.fa” is the output file
		
Convert that temporary file (i.e: “7864612_tmp.fa”) to a ‘.fa’ file using our “read_c_fa.cpp” file

   ./read_c_fa 7864612_tmp.fa 7864612.fa
	Here, 
	
“7864612_tmp.fa” is input file
“7864612.fa” is output file


Generating Clustal alignment:
----------------------------------
Please extract “poaV2.tar.gz” library or download the c implementation of POA library from here:
	https://sourceforge.net/projects/poamsa/
	
Compile the POA library via ‘make poa’ inside the directory.

Generate the Clustal file using ‘poa’ executable.

	./poa -read_fasta ../7864612.fa -clustal ../7864612_clustal.aln blosum80.mat -tolower
Here, 
“../7864612.fa” is input file 
“../7864612_clustal.aln” is output file

Generating the Consensus:
----------------------------------
Use our ‘.cpp’ implementation (i.e: “clustal_to_consensus_parsimony.cpp”) for generating the consensus.
	./clustal_to_consensus_parsimony 7864612_clustal.aln 7864612_clustal_consensus.fa
	Here,
		“7864612_clustal.aln” is clustal input file
		“7864612_clustal_consensus.fa” is final consensus fasta file

	Note: Please make sure that penalty matrix file (i.e: “score_matrix.txt”) is in the same directory of “./clustal_to_consensus_parsimony” executable.




B. Consensus generation using CCS
--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------


Downloading Modified Unanimity:
--------------------------------------------------------------------------

 git clone https://github.com/PacificBiosciences/unanimity 
  cd unanimity                                              && \
  git submodule update --init --remote                      && \
 

> rm -f ~/unanimity/src/ConsensusSettings.cpp

> rm -f ~/unanimity/src/poa/PoaGraphTraversals.cpp

> rm -f  ~/unanimity/include/pacbio/ccs/Consensus.h
> rm -f ~/unanimity/src/main/ccs.cpp

> cp ~/Improved-Consensus-for-Pacbio-Data/ConsensusUsingCCS/UnanimityChanges/ConsensusSettings.cpp  ~/unanimity/src/
> cp ~/PoaGraphTraversals.cpp  ~/unanimity/src/poa/
> cp ~/Consensus.h  ~/unanimity/include/pacbio/ccs/
> cp ~/ccs.cpp  ~/unanimity/src/main/

 mkdir build                                               && \
  cd build                                                  && \
  cmake ..                                                  && \
  make ccs                                                  && \
  ./ccs
  
  
 Issues Resolved in Unanimity Build in Development mode
---------------------------------------------------------------------
There was some compiling issue while making unanimity. It was not building libhts.a as dependency 

Resolve:
------------
> cd ~/unanimity/build/external/pbbam/build/external/htslib/
> make
(This will create libhts.a in ~/unanimity/build/external/pbbam/build/external/htslib/)
> cd ~/unanimity
> make ccs
 
 Running CCS:
 ---------------------------
./ccs --noPolish --containMult=2 --spanMult=1 --minCoverage=0  <input bam file>  <output bam file>

samtools view -h <output bam file>

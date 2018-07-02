# GET DATA: Get filtered fasta file from Cunning et al. 2015
# Cunning R, Yost DM, Guarinello ML, Putnam HM, Gates RD (2015) Data from: Variability 
#   of Symbiodinium communities in waters, sediments, and corals of thermally distinct 
#   reef pools in American Samoa. PLOS ONE https://doi.org/10.5061/dryad.32md8
data/fasta/combined_seqs_chimera_filtered.fasta:
	# Download data from Dryad
	wget https://datadryad.org/bitstream/handle/10255/dryad.102866/Cunning_PLoSONE_Dryad_Archive.zip
	# Extract fasta file and sample metadata
	unzip Cunning_PLoSONE_Dryad_Archive.zip AmSam.fasta sample_metadata.txt
	# Move and rename files for use with SymITS2 commands
	mv AmSam.fasta data/fasta/combined_seqs_chimera_filtered.fasta
	mv sample_metadata.txt data/mapping_file.txt
	# Delete archive
	rm Cunning_PLoSONE_Dryad_Archive.zip

# QC/TRIM READS
data/fasta/combined_seqs_trimmed.fasta: data/fasta/combined_seqs_chimera_filtered.fasta
	# Need to skip first 6 lines of script to begin with primer trimming
	tail -n +7 ~/SymITS2/qc_trim_reads.sh | bash
	# Trimming with SymITS2 script results in ~300k fewer sequences than Cunning et al. pipeline
	# replace '::' with '_' for compatibility with QIIME scripts
	sed -i '' 's/::/_/g' data/fasta/combined_seqs_trimmed.fasta

# CLUSTER WITHIN SAMPLES
data/clust/all_rep_set_rep_set.fasta: data/fasta/combined_seqs_trimmed.fasta
	bash ~/SymITS2/otus_97_bysample.sh data/fasta/combined_seqs_trimmed.fasta data/clust

# ASSIGN TAXONOMY TO REP SET SEQUENCES
data/clust/all_rep_set_rep_set_nw_tophits.tsv: data/clust/all_rep_set_rep_set.fasta
	R --vanilla < ~/SymITS2/run_nw.R --args data/clust/all_rep_set_rep_set.fasta data/ITS2db_trimmed_derep.fasta

# BUILD PHYLOSEQ
data/phy.RData: data/clust/all_rep_set_rep_set_nw_tophits.tsv
	R --vanilla < ~/SymITS2/build_phyloseq.R --args data/clust/all_rep_set_rep_set_nw_tophits.tsv data/mapping_file.txt data/clust/97_otus_bysample.tsv data/ITS2db_trimmed_notuniques_otus.txt data/phy.RData

# FILTER OUT NON-SYMBIODINIUM SEQUENCES
data/phy.f.RData: data/phy.RData
	R --vanilla < ~/SymITS2/filter_notsym.R --args data/phy.RData data/clust/all_rep_set_rep_set.fasta data/phy.f.RData $NT_PATH


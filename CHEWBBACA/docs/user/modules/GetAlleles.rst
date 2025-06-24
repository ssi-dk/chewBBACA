GetAlleles -  Create FASTA files containing the alleles identified by the AlleleCall module
===========================================================================================

The *GetAlleles* module is used to create FASTA files containing the alleles identified by the *AlleleCall* module. Given a TSV file with the allelic profiles, this module generates FASTA files for each gene/locus, including the nucleotide sequences and optionally the translated sequences (if the ``--translate`` option is provided). If the ``--distinct`` option is provided, the module will only include distinct alleles in the output files (this also applies to the FASTA files created when using the ``--translate`` option). If a TXT file with a list of gene/locus identifiers is provided through the ``--genes-list`` parameter, the module will only create FASTA files for the specified genes/loci. Otherwise, the module will create FASTA files for all genes/loci present in the input TSV file.

Basic Usage
-----------

::

	chewBBACA.py GetAlleles -i /path/to/AlleleCallResultsFolder/results_alleles.tsv -g /path/to/SchemaFolder -o /path/to/OutputFolder

Parameters
----------

::

    -i, --input-file            (Required) Path to the TSV file containing the allelic profiles (default: None).

    -g, --schema-directory      (Required) Path to the schema directory (default: None).

    --gl, --genes-list          (Optional) Path to a file with the list of genes/loci to create FASTA files for.
                                The file must include the identifiers of the loci, one per line, without the
                                .fasta extension (default: None).

    -o, --output-directory      (Required) Path to the output directory (default: None).

    --cpu, --cpu-cores          (Optional) Number of CPU cores/threads that will be used to run the process (chewie
                                resets to a lower value if it is equal to or exceeds the total number of
                                available CPU cores/threads) (default: 1).

    --distinct                  (Optional) Only get distinct alleles (default: False).


    --translate                 (Optional) Create FASTA files with the translated alleles (default: False).

    --ta, --translation-table   (Optional) Genetic code used to translate coding DNAsequences (CDSs). If no value
                                is specified, the process tries to get the value stored in the schema
                                config file. If the schema does not include a config file, the process
                                uses the default translation table (11) (default: None).

Outputs
:::::::

::

   OutputFolderName
   ├── fastas
   │   ├── locus1.fasta
   │   ├── ...
   │   └── locusN.fasta
   ├── translated_fastas
   │   ├── locus1_protein.fasta
   │   ├── ...
   │   └── locusN_protein.fasta
   └── summary_stats.tsv

- The ``fastas`` folder contains the FASTA files for each gene/locus, named as ``locusID.fasta``, where *locusID* is the identifier of the locus. Each FASTA file includes the nucleotide sequences of the alleles identified in all strains listed in the input file. The header of each sequence includes the strain, locus, and allele identifier (e.g. ``>strainID_locusID_alleleID``). If the ``--distinct`` option is provided, only distinct alleles will be included in the FASTA files and the header will only include the locus and allele identifier (e.g. ``>locusID_alleleID``).

- The ``translated_fastas`` folder contains the FASTA files for each gene/locus with the translated sequences, named as ``locusID_protein.fasta``. Each FASTA file includes the protein sequences of the alleles identified in all strains listed in the input file. The sequence headers follow the same format as the one used for the nucleotide sequences.

- The ``summary_stats.tsv`` file contains, for each locus, the number of alleles in the schema, the number of samples in which the locus was identified, and the number of distinct alleles identified in the dataset.

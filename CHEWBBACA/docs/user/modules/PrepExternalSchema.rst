PrepExternalSchema - Adapt an external schema to be used with chewBBACA
=======================================================================

The PrepExternalSchema module enables the adaptation of external schemas so that it is possible to use those schemas with chewBBACA. An external schema may be a set of sequences from any number of genes that have been selected for a particular study or it may be a schema that has already been defined and is available for download from some well known databases, such as:

	- `Ridom cgMLST <http://www.cgmlst.org/ncs>`_.
	- `BIGSdb <https://pubmlst.org/>`_.
	- `BIGSdb-Pasteur <https://bigsdb.pasteur.fr/>`_.
	- `Enterobase <http://enterobase.warwick.ac.uk/>`_.

External schemas are defined with specific parameters that might differ from the parameters and conditions enforced by chewBBACA. Therefore, these external schemas need to be processed to filter out sequences that do not meet a set of criteria applied to create every chewBBACA schema. Every sequence that is included in the final schema has to represent a complete coding sequence (the first and last codons must be valid start and stop codons, the sequence length must be a multiple of 3 and cannot contain in-frame stop codons) and contain no invalid or ambiguous characters (sequences must be composed of ATCG only).

Basic Usage
-----------

To adapt an external schema, the FASTA files with the allele sequences for each gene of the schema have to be in a single directory. Alternatively, a file containing the list of paths to the schema files may also be provided.

::

	chewBBACA.py PrepExternalSchema -g /path/to/ExternalSchemaFolder -o /path/to/OutputFolder --ptf /path/to/ProdigalTrainingFile

Parameters
----------

::

    -g, --schema-directory     (Required) Path to the directory of the schema to adapt. The schema must contain
                               one FASTA file per gene/locus.

    -o, --output-directory     (Required) Path to the output directory where the adapted schema will be created.

    --gl, --genes-list         (Optional) Path to a file with the list of loci in the schema that the process
                               should adapt (one per line, full paths or loci IDs) (default: False).

    --ptf, --training-file     (Optional) Path to the Prodigal training file that will be included in the
                               directory of the adapted schema (default: None).

    --bsr, --blast-score-ratio (Optional) BLAST Score Ratio (BSR) value. The process selects representative
                               alleles for each locus based on this value. Representative alleles are
                               selected until all alleles in a locus align against one of the representatives
                               with a BSR >= than the specified value (default: 0.6).

    --l, --minimum-length      (Optional) Minimum sequence length value stored in the schema config file. The
                               schema adaptation process will only discard sequences smaller than this value
                               if the --size-filter parameter is provided (default: 0).

    --t, --translation-table   (Optional) Genetic code used for allele translation (default: 11).

    --st, --size-threshold     (Optional) Allele size variation threshold value stored in the schema config file.
                               The schema adaptation process will only discard alleles with a size that deviates
                               from the locus length mode +- the size theshold value if the --size-filter parameter
                               is provided(default: 0.2).

    --cpu, --cpu-cores         (Optional) TNumber of CPU cores/threads that will be used to run the process
                               (chewie resets to a lower value if it is equal to or exceeds the total number
                               of available CPU cores/threads) (default: 1).

    --b, --blast-path          (Optional) Path to the directory that contains the BLAST executables
                               (default: assumes BLAST executables were added to PATH).

    --size-filter              (Optional) Apply the minimum length and size threshold values to
                               filter out alleles during schema adaptation (default: False).

Outputs
-------

- ``<adapted_schema>_invalid_alleles.txt`` - contains the identifiers of the alleles that were excluded and the reason for the exclusion of each allele.
- ``<adapted_schema>_invalid_genes.txt`` - contains the list of genes that had no valid alleles, one gene identifier per line.
- ``<adapted_schema>_summary_stats.tsv`` - contains summary statistics for each gene (number of alleles in the external schema, number of valid alleles included in the adapted schema and number of representative alleles chosen by chewBBACA).

.. note::
	For most genes, only one or a few sequences need to be chosen as representatives to represent the gene sequence diversity. Nevertheless, some genes will have a high number of representatives. This is more common for small genes, where a small number of differences has a big impact on the alignment score, for genes with repetitive or low complexity regions that may be masked by BLAST and lead to lower alignment scores between highly similar sequences, and for genes that have inversions, deletions or insertions that can lead to several High-scoring Segment Pairs (HSPs), none of which have a score sufficiently high to identify both sequences as belonging to the same gene.

Workflow of the PrepExternalSchema module
:::::::::::::::::::::::::::::::::::::::::

.. image:: /_static/images/PrepExternalSchema.png
   :width: 1000px
   :align: center

The PrepExternalSchema module adapts schemas created with other wg/cgMLST tools or available on external platforms for usage with chewBBACA 3. Brief description of the workflow:

- The process starts by validating and translating the alleles in the external schema. Incomplete (i.e. size not multiple of 3) and invalid (i.e. missing the start or stop codons, or containing in-frame stop codons) alleles, alleles containing ambiguous bases or smaller than the specified minimum length value, are excluded.

- For each locus that has valid alleles, the process selects the largest or one of the largest alleles as the first representative allele.

- The representative is aligned against the locus' alleles with BLASTp to compute the BSR for each alignment. If all the BSR values are above the specified BSR (default is 0.6) plus 0.1, it is considered that the representative allele can adequately capture the diversity of the locus.

- Otherwise, new representative alleles are selected from those with a BSR above the specified BSR but below that value plus 0.1 to align against the locus' alleles and determine if the set of representative alleles selected captures the locus diversity adequately.

- Representative selection is repeated until all locus' alleles have a BSR above the specified value plus 0.1 with at least one of the selected representative alleles.

- The valid and selected representative alleles are written to FASTA files to create a schema compatible with chewBBACA. The list of invalid alleles, the list of loci excluded from the adapted schema due to having no valid alleles, and the number of total alleles and representative alleles per locus in the adapted schema are stored in output files.

UniprotFinder - Retrieve annotations for loci in a schema
=========================================================

The UniprotFinder module retrieves annotations for the loci in a schema through requests to `UniProt's SPARQL endpoint <http://sparql.uniprot.org/sparql>`_ and through alignment with BLASTp against UniProt's reference proteomes for a set of taxa.

Basic Usage
-----------

::

	chewBBACA.py UniprotFinder -g /path/to/SchemaFolder -o /path/to/OutputFolder -t /path/to/cds_coordinates.tsv --taxa "Species Name" --cpu 4

Parameters
----------

::

    -g, --schema-directory (Required) Path to the schema's directory.

    -o, --output-directory (Required) Output directory where the process will store intermediate
                           files and save the final TSV file with the loci annotations.

    --gl, --genes-list     (Optional) Path to a file with the list of loci in the schema that the process should
                           find annotations for (one per line, full paths or loci IDs) (default: False).

    -t, --protein-table    (Optional) Path to the TSV file with coding sequence (CDS) coordinate data,
                           "cds_coordinates.tsv", created by the CreateSchema process. (default: None).

    --bsr                  (Optional) BLAST Score Ratio value. The BSR is only used when taxa names are provided
                           to the --taxa parameter and local sequences are aligned against reference
                           proteomes downloaded from UniProt. Annotations are selected based on a BSR
                           >= than the specified value (default: 0.6).

    --cpu, --cpu-cores     (Optional) Number of CPU cores/threads that will be used to run the process
                           (chewie resets to a lower value if it is equal to or exceeds the total number
                           of available CPU cores/threads) (default: 1).

    --taxa                 (Optional) List of scientific names for a set of taxa. The process will download
                           reference proteomes from UniProt associated to taxa names that contain any
                           of the provided terms. The schema representative alleles are aligned
                           against the reference proteomes to assign annotations based on high-BSR matches
                           (default: None).

    --pm                   (Optional) Maximum number of proteome matches to report (default: 1).

    --no-sparql            (Optional) Do not search for annotations through the UniProt SPARQL endpoint.

    --no-cleanup           (Optional) If provided, intermediate files generated during process execution are not
                           removed at the end (default: False).

    --b, --blast-path      (Optional) Path to the directory that contains the BLAST executables.

Outputs
-------

The process writes a TSV file, ``schema_annotations.tsv``, with the annotations found for each locus in the directory passed to the ``-o`` parameter. By default, the process searches for annotations through UniProt's SPARQL endpoint, reporting the product name and UniProt URL for local loci with an exact match in UniProt's database. If the ``cds_coordinates.tsv`` file is passed to the ``-t`` parameter, the output file will also include the loci coordinates. The ``--taxa`` parameter receives a set of taxa names and searches for reference proteomes that match the provided terms. The reference proteomes are downloaded and the process aligns the loci representative alleles against the reference proteomes to include the product and gene name from the reference proteomes in the output file.

Workflow of the UniprotFinder module
::::::::::::::::::::::::::::::::::::

.. image:: /_static/images/UniprotFinder.png
   :width: 1000px
   :align: center

The UniprotFinder module determines annotations for schema loci. Brief description of the workflow:

	- The module offers two options to determine annotations: aligning against UniProt's reference proteomes and exact matching through UniProt's SPARQL endpoint.

	- Users must provide at least one valid taxon name to annotate based on the reference proteomes. The process downloads the list of reference proteomes and searches for proteomes for the specified taxa. If there are any proteomes for the specified taxa, they are downloaded, and the loci representative alleles are aligned against the reference proteomes so annotations can be selected based on the BSR.

	- The process searches for annotations through UniProt's SPARQL endpoint by creating queries including the loci alleles and submitting requests to the endpoint. If an allele matches any protein in UniProt, the annotation terms are extracted from the results. The process tries to select the most informative annotation terms.

	- The annotation terms found through both options are merged to create a single annotations table. If the user provides a TSV file with additional loci data, such as the file with CDS coordinates created by the CreateSchema and AlleleCall modules, the process will add the data in that file to the annotations table.

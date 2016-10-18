---
layout:     page
title:      Parsing BLAST XML output using Bio.SearchIO 
date:       2016-10-05
summary:    Test Post
categories: Test
---

The package `SearchIO` from Biopython parses outputs from BLAST and other sequence search programs; `SearchIO` will eventually replace [`Bio.Blast.NCBIXML`](http://biopython.org/DIST/docs/api/Bio.Blast.NCBIXML-module.html). Further detail about `SearchIO` is available in its [official documentation](http://biopython.org/DIST/docs/api/Bio.SearchIO-module.html) and on [Biopython's wiki](http://biopython.org/wiki/SearchIO).

In this example, I use `SearchIO` to parse BLAST output in XML format and extract specific contents. The correspondence between elements in the BLAST XML output and attributes for each `SearchIO` object is available [here](http://biopython.org/DIST/docs/api/Bio.SearchIO.BlastIO-module.html). 

```python
#!/usr/bin/env python

import sys

from Bio import SearchIO

"""
Create a tab-delimited text file, parse one or more BLAST XML files, and 
print information from BLAST search output to text file

Usage: python parse_blast_xml.py outfile.txt blastoutput.xml
"""

out = open(sys.argv[1], 'w')
out.write("Query Name\tQuery Definition\tQuery Length\tHit ID\tHit Defintion\tHit Length\teValue\n")
for xml_file in sys.argv[2:]:
    result_handle = open(xml_file)
    qresults = SearchIO.parse(result_handle, 'blast-xml')
    for qres in qresults:
        for hit in qres.hits:
            for hsp in hit.hsps:
                fields = [qres.id, qres.description, str(qres.seq_len), hit.id, hit.description, 
                str(hit.seq_len), str(hsp.evalue)]
                out.write("\t".join(fields) + "\n")
out.close()
```
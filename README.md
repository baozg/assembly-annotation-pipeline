# Snakemake workflow: plant genome HiFi assembly and genome annotation

This Snakemake pipeline aims for plant genome assembly with HiFi data and genome annotation with RNA-Seq/IsoSeq.

> [!WARNING]  
> This is not up-to-date pipeline. Lots of module could be replaced by modern tools. For example, de novo annotation from augustus and other tools could replaced by helixer and ANNEVO. MAKER2/3 is very hard for running in parrlel, EviAnn or EGAPx would be good canidate.

## Assembly

Modified from the VGP assembly [VGP tutorial](https://training.galaxyproject.org/training-material/topics/assembly/tutorials/vgp_genome_assembly/tutorial.html)

![Assembly](./images/Assembly.png)




## Annotation


- Maker running inside `snakemake` still have the problem since the NSF lock and submit system is already included.
- IsoSeq integration is still in development. For diploid or polyploidy, FLNC mapping is better than the clustering of the PacBio pipeline if you have haplotype-resolved assembly.
- Automatic pipeline always confused the complex gene clusters (metabolic gene clusters / NLR gene clusters), please manually curation it before any further interpretation. `WebApollo` or `IGV-GSAman` could be a good start for manual curation. (https://www.bilibili.com/video/BV1x84y1z7ZW/)


![Annotation](./images/Annotation.png)


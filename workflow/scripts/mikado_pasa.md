mikado_pasa (examples from tetraploid potato)

# maker

# portcullis (high confidence splice junction)
```bash
portcullis prep -t 64 ../../01.ref/C88.V1.fa C88.V1.S10.sorted.bam C88.V1.S11.sorted.bam C88.V1.S12.sorted.bam C88.V1.S13.sorted.bam C88.V1.S14.sorted.bam C88.V1.S15.sorted.bam C88.V1.S16.sorted.bam C88.V1.S17.sorted.bam C88.V1.S18.sorted.bam C88.V1.S19.sorted.bam C88.V1.S1.sorted.bam C88.V1.S20.sorted.bam C88.V1.S21.sorted.bam C88.V1.S22.sorted.bam C88.V1.S2.sorted.bam C88.V1.S3.sorted.bam C88.V1.S4.sorted.bam C88.V1.S6.sorted.bam C88.V1.S7.sorted.bam C88.V1.S8.sorted.bam C88.V1.S9.sorted.bam
portcullis junc -t 64 ./portcullis_prep
portcullis filter -t 64 portcullis_prep ./portcullis_junc/portcullis.junctions.tab
```

# mikado
=====list.txt (stringtie + IsoSeq gff3)==========
C88.V1.S10.gtf  C88.V1.S10      True    1       False   True    True
C88.V1.S11.gtf  C88.V1.S11      True    1       False   True    True
C88.V1.S12.gtf  C88.V1.S12      True    1       False   True    True
C88.V1.S13.gtf  C88.V1.S13      True    1       False   True    True
C88.V1.S14.gtf  C88.V1.S14      True    1       False   True    True
C88.V1.S15.gtf  C88.V1.S15      True    1       False   True    True
C88.V1.S16.gtf  C88.V1.S16      True    1       False   True    True
C88.V1.S17.gtf  C88.V1.S17      True    1       False   True    True
C88.V1.S18.gtf  C88.V1.S18      True    1       False   True    True
C88.V1.S19.gtf  C88.V1.S19      True    1       False   True    True
C88.V1.S1.gtf   C88.V1.S1       True    1       False   True    True
C88.V1.S20.gtf  C88.V1.S20      True    1       False   True    True
C88.V1.S21.gtf  C88.V1.S21      True    1       False   True    True
C88.V1.S22.gtf  C88.V1.S22      True    1       False   True    True
C88.V1.S2.gtf   C88.V1.S2       True    1       False   True    True
C88.V1.S3.gtf   C88.V1.S3       True    1       False   True    True
C88.V1.S4.gtf   C88.V1.S4       True    1       False   True    True
C88.V1.S6.gtf   C88.V1.S6       True    1       False   True    True
C88.V1.S7.gtf   C88.V1.S7       True    1       False   True    True
C88.V1.S8.gtf   C88.V1.S8       True    1       False   True    True
C88.V1.S9.gtf   C88.V1.S9       True    1       False   True    True
isoseq.gff3     pb      True    2       False   False   False

======
```bash
mikado configure --list list.txt --reference C88.V1.fa --mode permissive --scoring plant.yaml --copy-scoring plant.yaml -bt uniprot_sprot_plants.fa --junctions portcullis.pass.junctions.bed configuration.yaml
mikado prepare --json configuration.yaml
diamond makedb --in uniprot_sprot_plants.fa --db unipro
diamond blastx --max-target-seqs 5 --outfmt 6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore ppos btop -q ../mikado_prepared.fasta -d uniprot -o mikado_prepared.blast.tsv -p 64
```
## transdecoder

```bash
for i in `seq -w 1 64`;
  do 
    ~/software/TransDecoder/5.5.0/TransDecoder.LongOrfs -t split/mikado_prepared.part_043.fasta
    diamond blastp --max-target-seqs 1 --outfmt 6 --evalue 1e-5 -p 1 -q mikado_prepared.part_${i}.fasta.transdecoder_dir/longest_orfs.pep -d uniprot > mikado_prepared.part_${i}.fasta.transdecoder_dir/blastp.tsv
    ~/software/InterProscan/interproscan-5.48-83.0/bin/hmmer/hmmer3/3.1b1/hmmscan --cpu 1 --domtblout mikado_prepared.part_${i}.fasta.transdecoder_dir/pfam.domtblout ~/software/InterProscan/interproscan-5.48-83.0/data/pfam/33.1/pfam_a.hmm ./mikado_prepared.part_${i}.fasta.transdecoder_dir/longest_orfs.pep
    ~/software/TransDecoder/5.5.0/TransDecoder.Predict -t split/mikado_prepared.part_${i}.fasta --retain_pfam_hits mikado_prepared.part_${i}.fasta.transdecoder_dir/pfam/domtblout --retain_blastp_hist mikado_prepared.part_${i}.fasta.transdecoder_dir/blastp.tsv
done
```
# mikado
```bash
mikado serialise -p 48 --json-conf configuration.yaml --xml 01.blast/mikado_prepared.blast.tsv --orfs mikado_prepared.transdecoder.orfs.gff3
mikado pick -p 48 --json-conf configuration.yaml --subloci-out mikado.subloci.gff3
```

```bash
## pasa (2.5.1)
export PASAHOME=/public10/home/sci0009/software/PASApipeline/2.5.1/

# mikado fasta (use cdna for UTR)
gffread -x mikado.loci.cds -y mikado.pep.fa -w mikado.cdna.fa -g C88.V1.fa mikado.loci.gff3

# combine isoseq and mikado
cat isoseq.hq.fasta mikado.cdna.fa > C88.mikado.isoseq.fa
seqclean  transcripts.fasta -v /path/to/your/vectors.fasta

### align (-fl for iso-seq hq cluster name)
/public10/home/sci0009/software/PASApipeline/2.5.1/Launch_PASA_pipeline.pl -c alignAssembly.config -C -R -g C88.V1.fa -f fl.list -t C88.isoseq.mikado.fa.clean -T -u C88.isoseq.mikado.fa --ALIGNERS gmap,minimap2 --CPU 64

### load maker gff3
/public10/home/sci0009/software/PASApipeline/2.5.1/scripts/Load_Current_Gene_Annotations.dbi -c alignAssembly.config -g C88.V1.fa -P C88.aed.filter.gtf

### compare and update (1~2 day)
/public10/home/sci0009/software/PASApipeline/2.5.1/Launch_PASA_pipeline.pl -c annotCompare.config -A -g C88.V1.fa -t C88.isoseq.mikado.fa --CPU 64
```

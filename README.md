# NovaSeq X demultiplexing — OverrideCycles & SampleSheets

Reference and tooling for demultiplexing Illumina NovaSeq X runs with **bcl-convert**, focused on CUT&Tag and CUT5base (easySci) libraries. Includes a browser tool that builds the `OverrideCycles` string, ready-to-edit SampleSheet templates, and the run commands.

## Contents

- [`index.html`](index.html) — OverrideCycles Builder (web app, no dependencies)
- OverrideCycles cheat sheet — [short reads](#short-reads-2--61-cycle) and [long reads](#long-reads-2--134-cycle)
- SampleSheet templates for each library type
- [Running bcl-convert](#running-bcl-convert) — direct command and SLURM wrapper

## The OverrideCycles Builder

Open [`index.html`](index.html) in any browser. Enter the run's four cycle counts, then for each read fill any of **Y / I / N / U** with a length and a 1-based start position. Uncovered cycles are padded with `N`, and Index 2 (i5) is auto-reversed to match NovaSeq X's reverse-complement read.

| Letter | Meaning |
|---|---|
| `Y` | sequence, output to FASTQ |
| `I` | index, used for demultiplexing |
| `N` | masked / discarded cycles |
| `U` | UMI, trimmed into the FASTQ header |

Two builds exist historically: `overridecycles_builder_V1_autorefresh.html` (live update) and `overridecycles_builder_V2_generator.html` (explicit Generate button). The current `index.html` is the generator build.

## OverrideCycles cheat sheet

Segment order is `Read1 ; Index1(i7) ; Index2(i5) ; Read2` and is **position-sensitive** — `I8N2` (index at cycle 1) is not the same as `N2I8` (index at cycle 3). Each read's tokens must sum to its full cycle count.

### Short reads (2 × 61 cycle)

Run structure: Read1 = 57, Read2 = 61, Index1 = 10, Index2 = 10.

| Library type | OverrideCycles | Notes |
|---|---|---|
| General CUT&Tag (dual index) | `Y57;I8N2;N2I8;Y61` | 8 bp index, i5 index sits at cycle 3 |
| CUT5base / easySci (i7-only) | `Y57;I8N2;U10;Y61` | i5 read taken as 10 bp UMI |

### Long reads (2 × 134 cycle)

Run structure: Read1 = 134, Read2 = 134, Index1 = 10, Index2 = 60.

| Library type | OverrideCycles | Notes |
|---|---|---|
| General CUT&Tag / multiome (dual index) | `Y50N84;I8N2;N52I8;Y71N63` | see breakdown below |
| CUT5base (i7-only) | `Y134;I8N2;N50U10;Y134` | i5 read: discard 50, keep 10 as UMI |

Breakdown of `Y50N84;I8N2;N52I8;Y71N63`:
- Read1 — keep first 50 bp, discard last 84
- Index1 (i7) — keep 8 bp, discard last 2
- Index2 (i5) — discard first 52 bp, keep last 8 as index
- Read2 — keep first 71 bp, discard last 63

## SampleSheet templates

Each block below goes in a `.csv`. Fill `Index` / `Index2` per sample. `NA` in Index2 means i5 is not used for demultiplexing.

### General CUT&Tag, short reads (dual index)

```
[Header]
FileFormatVersion,2
RunName,RL
RunDescription,CUT&Tag
InstrumentPlatform,NovaSeqXSeries
IndexOrientation,Forward

[Reads]
Read1Cycles,57
Read2Cycles,61
Index1Cycles,10
Index2Cycles,10

[BCLConvert_Settings]
SoftwareVersion,4.1.7
BarcodeMismatchesIndex1,0
BarcodeMismatchesIndex2,0
FastqCompressionFormat,gzip

[BCLConvert_Data]
Sample_ID,Index,Index2,OverrideCycles,Sample_Project
RL8264_..._shear_control_lamdaDNA_PUC19,TAAGGCGA,CGTCTAAT,Y57;I8N2;N2I8;Y61,chromatin_remodeling_CUTnTag
RL8311_..._H3K4me2_spikein,AACGGGTG,CTCTCTAT,Y57;I8N2;N2I8;Y61,chromatin_remodeling_CUTnTag
```

### CUT5base / easySci, short reads (i7-only demux)

Index 2 is not declared; i5 cycles are taken as a UMI.

```
[Header]
FileFormatVersion,2
RunName,S587
RunDescription,CUT&Tag
InstrumentPlatform,NovaSeqXSeries
IndexOrientation,Forward

[Reads]
Read1Cycles,57
Read2Cycles,61
Index1Cycles,10

[BCLConvert_Settings]
SoftwareVersion,4.1.7
BarcodeMismatchesIndex1,0
CreateFastqForIndexReads,1
TrimUMI,0
FastqCompressionFormat,gzip

[BCLConvert_Data]
Sample_ID,Index,Index2,OverrideCycles,Sample_Project
RL8262_..._CUTnTagno5base_2ndTn5,CGTACTAG,NA,Y57;I8N2;U10;Y61,CUT5base
RL8263_..._CUTnTag5base_spikincontrol,AGGCAGAA,NA,Y57;I8N2;U10;Y61,CUT5base
```

### General multiome / methylome, long reads (dual index, per-lane)

```
[Header]
FileFormatVersion,2
RunName,10B_5Base_methylomes_..._HG_DL_DM_OC
RunDescription,5base
InstrumentPlatform,NovaSeqXSeries
IndexOrientation,Forward

[Reads]
Read1Cycles,134
Read2Cycles,134
Index1Cycles,10
Index2Cycles,60

[BCLConvert_Settings]
BarcodeMismatchesIndex1,0
FastqCompressionFormat,gzip

[BCLConvert_Data]
Lane,Sample_ID,Index,Index2,OverrideCycles,Sample_Project
6,RL8461_..._multiome_Rhapsody_GEX,CGGAGCCT,ATAGAGGC,Y50N84;I8N2;N52I8;Y71N63,Engram
6,RL8462_..._multiome_Rhapsody_HTOs,CGGCTATG,CAGGACGT,Y50N84;I8N2;N52I8;Y71N63,Engram
```

### CUT5base, long reads (i7-only demux)

Index 2 is not declared; i5 read: discard 50 bp, keep last 10 as UMI.

```
[Header]
FileFormatVersion,2
RunName,CUT5baseHG
RunDescription,CUT&Tag
InstrumentPlatform,NovaSeqXSeries
IndexOrientation,Forward

[Reads]
Read1Cycles,134
Read2Cycles,134
Index1Cycles,10

[BCLConvert_Settings]
SoftwareVersion,4.1.7
BarcodeMismatchesIndex1,0
CreateFastqForIndexReads,1
TrimUMI,0
FastqCompressionFormat,gzip

[BCLConvert_Data]
Sample_ID,Index,Index2,OverrideCycles,Sample_Project
RL8472_..._CUTnTagno5base_H3K4me1,CGTACTAG,NA,Y134;I8N2;N50U10;Y134,CUT5base_HG
RL8473_..._CUTnTagno5base_H3K27ac,AGGCAGAA,NA,Y134;I8N2;N50U10;Y134,CUT5base_HG
```

## Running bcl-convert

### Direct command

```bash
bcl-convert \
  --bcl-input-directory /group/llshared/sequencing_data/NovaSeqX/<RUN_FOLDER> \
  --output-directory    /group/llshared/sequencing_data/NovaSeqX/<RUN_FOLDER>/fastqs/ \
  --bcl-sampleproject-subdirectories true \
  --no-lane-splitting true \
  --force \
  --sample-sheet /path/to/SampleSheet.csv
```

### SLURM wrapper

```bash
sbatch /group/ll010/hgao/demultiplex/demux_run.slurm \
  /group/llshared/sequencing_data/NovaSeqX/<RUN_FOLDER> \
  1 \
  /group/ll010/hgao/demultiplex/Sample_sheet_260716_CUT5base_seq.csv
```

Positional arguments: (1) run folder, (2) barcode mismatch allowance — `1` allows one mismatched base, (3) SampleSheet path.

## Notes

- On NovaSeq X, Index 2 (i5) is read reverse-complement (`IsReverseComplement=Y` in `RunInfo.xml`). Fill i5 in physical order in the web app; tokens are reversed for you.
- If Index 2 keeps any `I` cycles, the SampleSheet's `[BCLConvert_Data]` must include an `Index2` column. For i7-only demux, drop `Index2Cycles` from `[Reads]` and mask i5 in OverrideCycles.
- Cycle counts should always match the run's `RunInfo.xml`.

## License

MIT — see [LICENSE](LICENSE).

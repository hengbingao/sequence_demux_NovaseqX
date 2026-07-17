# OverrideCycles Builder

A single-page, dependency-free web tool for building the `OverrideCycles` string used by Illumina **bcl-convert** on NovaSeq X runs.

You enter the run's cycle structure and, for each read, fill any of **Y / I / N / U** with a length and a start position. The tool places the segments, pads gaps with `N`, and writes the string (e.g. `Y134;I8N2;N50U10;Y134`).

## Why

Getting `OverrideCycles` right is fiddly: segment order is position-sensitive (`I8N2` ≠ `N2I8`), i5 (Index 2) is read reverse-complement on NovaSeq X, and each read's tokens must sum to its full cycle count. This tool makes those constraints visible and hard to get wrong.

## Features

- **Four cycle inputs** — Read1 / Index1 (i7) / Index2 (i5) / Read2, taken from `RunInfo.xml`.
- **Per-read Y / I / N / U segments** — each with a length and a 1-based start position; leave a row blank to skip it.
- **Automatic N padding** — any cycle not covered by a segment becomes `N`.
- **i5 auto-reverse** — fill i5 in physical read order; the tool reverses its tokens to match bcl-convert's reverse-complement convention. The ruler shows what you typed; the output shows what bcl-convert needs.
- **Live rulers** — a colored bar per read shows the Y/I/N/U layout as you type.
- **Overlap & overflow detection** — warns if two segments claim the same cycle or a segment runs past the read length.
- **Generate / Regenerate** — build the string on demand; **Copy string** copies the current value.

## Letter meanings

| Letter | Meaning |
|---|---|
| `Y` | sequence, output to FASTQ |
| `I` | index, used for demultiplexing |
| `N` | masked / discarded cycles |
| `U` | UMI, trimmed into the FASTQ header |

## Usage

Open `index.html` in any modern browser. No build step, no dependencies, no network calls.

### Example — `Y134;I8N2;N50U10;Y134`

Run structure: Read1 = 134, Index1 = 10, Index2 = 60, Read2 = 134.

| Read | Row | Length | Start |
|---|---|---|---|
| Read 1 | Y | 134 | 1 |
| Index 1 (i7) | I | 8 | 1 |
| Index 2 (i5) | U | 10 | 1 |
| Read 2 | Y | 134 | 1 |

i7 keeps 8 bp of index and auto-pads `N2`. i5 is filled in physical order (UMI first) and auto-reversed to `N50U10`. Click **Generate**.

## Notes on i5 (Index 2)

On NovaSeq X, Index 2 is read reverse-complement (`IsIndexedRead=Y`, `IsReverseComplement=Y` in `RunInfo.xml`). Fill i5 in the order you physically read it; the emitted tokens are reversed for you. If i5 keeps any `I` cycles, your SampleSheet's `[BCLConvert_Data]` section needs an `Index2` column.

## License

MIT — see [LICENSE](LICENSE).

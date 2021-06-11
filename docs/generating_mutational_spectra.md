## Generating Mutational Spectra

Generating mutational signatures using  `signatureanalyzer`. For a comprehensive  description of mutational signatures, their relevance, and references, please see the Catalogue of Somatic Mutations in Cancer, or COSMIC, [here](https://cancer.sanger.ac.uk/cosmic/signatures), as well as *The repretoire of mutational signatures in human cancer*(https://www.nature.com/articles/s41586-020-1943-3). The following document is a reference for easy creation of spectra using `.mafs`.

---

#### Load Mutational Annotation File (`maf`)

For more information about the `.maf` format, see [here](https://docs.gdc.cancer.gov/Data/File_Formats/MAF_Format/). The following examples show generating mutational spectra from an `hg19` `.maf` using `signatureanalyzer`. First, load your `maf` as a pandas dataframe.

The following columns are **required**:
* `Tumor_Sample_Barcode`
* `Chromosome`
* `Start_Position`
* `Reference_Allele`
* `Tumor_Seq_Allele2`
* `Variant_Type`

The following columns are **optional**:
* `ref_context`: will map context if not provided

```{python}
import signatureanalyzer as sa
import pandas as pd

maf_df = pd.read_csv(<PATH_TO_MAF>, sep='\t').loc[:,[
  'Hugo_Symbol',
  'Tumor_Sample_Barcode',
  'Chromosome',
  'Start_Position',
  'Reference_Allele',
  'Tumor_Seq_Allele2',
  'Variant_Type'
  ]]

print(maf_df.head())
```
|    | Hugo_Symbol   | Tumor_Sample_Barcode   |   Chromosome |   Start_Position | Reference_Allele   | Tumor_Seq_Allele2   | Variant_Type   |
|---:|:--------------|:-----------------------|-------------:|-----------------:|:-------------------|:--------------------|:---------------|
|  0 | URGCP         | sample_192             |            7 |         43916856 | C                  | A                   | SNP            |
|  1 | CLCN1         | sample_127             |            7 |        143048832 | C                  | A                   | SNP            |
|  2 | NAV2          | sample_354             |           11 |         20113762 | A                  | G                   | SNP            |
|  3 | TUBB8P7       | sample_32              |           16 |         90162224 | G                  | T                   | SNP            |
|  ... | CHFR          | sample_35              |           12 |        133438086 | C                  | A                   | SNP            |

---

#### Single-base-substitution (SBS) spectra
* This encodes the 96-base context
* **note**: two forms of this exist - either input should work
  * word: ACAG --> (REF)(MUT)(LEFT NT)(RIGHT NT)
  * arrow: A[A>C]G --> (LEFT NT)(REF)>(MUT)(RIGHT NT)
* REQUIRES a 2-bit human genome build

```
_,spectra_sbs = sa.spectra.get_spectra_from_maf(maf_df, reference='cosmic3_exome', hgfile='hg19.2bit')

print(spectra_sbs.head().iloc[:,:5])
```
**Arrow**
|         |   sample_551 |   sample_135 |   sample_118 |   sample_191 |   sample_124 |
|:--------|-------------:|-------------:|-------------:|-------------:|-------------:|
| A[A>C]A |            1 |            2 |            0 |            1 |            2 |
| A[A>C]C |            0 |            2 |            0 |            0 |            0 |
| A[A>C]G |            0 |            6 |            2 |            2 |            0 |
| A[A>C]T |            0 |            4 |            0 |            0 |            2 |
| ... |            1 |            2 |            0 |            1 |            0 |

_or_

**Word**
| context96.word   |   sample_551 |   sample_135 |   sample_118 |   sample_191 |   sample_124 |
|:-----------------|-------------:|-------------:|-------------:|-------------:|-------------:|
| ACAA             |            1 |            2 |            0 |            1 |            2 |
| ACAC             |            0 |            2 |            0 |            0 |            0 |
| ACAG             |            0 |            6 |            2 |            2 |            0 |
| ACAT             |            0 |            4 |            0 |            0 |            2 |
| ...             |            1 |            2 |            0 |            1 |            0 |
---

#### Doublet-base-substitution (DBS) spectra
* This encodes the 78-base context

```
_,spectra_dbs = sa.spectra.get_spectra_from_maf(maf_df, reference='cosmic3_DBS')

print(spectra_dbs.head())
```

| context78.word   |   sample_0 |   sample_1 |   sample_2 |   sample_3 |   sample_4 |
|:-----------------|-----------:|-----------:|-----------:|-----------:|-----------:|
| AC>CA            |          0 |          0 |          0 |          0 |          0 |
| AC>CG            |          0 |          0 |          0 |          0 |          0 |
| AC>CT            |          0 |          0 |          0 |          0 |          0 |
| AC>GA            |          0 |          0 |          0 |          0 |          0 |
| ...            |          0 |          0 |          0 |          0 |          0 |

---

#### Insertions & Deletions (ID) spectra
* This encodes the 83-base context
* REQUIRES a 2-bit human genome build

```
_,spectra_id = sa.spectra.get_spectra_from_maf(maf_df, reference='cosmic3_ID', hgfile='hg19.2bit')

print(spectra_id.head().iloc[:,:5])
```

| context83.word   |   sample_0 |   sample_1 |   sample_2 |   sample_3 |   sample_4 |
|:-----------------|-----------:|-----------:|-----------:|-----------:|-----------:|
| Cdel1            |          0 |          1 |          3 |          1 |          0 |
| Cdel2            |          0 |          4 |          3 |          6 |          0 |
| Cdel3            |          2 |          2 |          2 |          9 |          0 |
| Cdel4            |          0 |          0 |          0 |          3 |          0 |
| ...            |          0 |          0 |          1 |          3 |          0 |

---

#### Single-base-substitution (SBS) 1536-pentanucleotide spectra
* This encodes the 1536-base context
* **note**: two forms of this exist - either input should work
  * word: ACAGAT --> (REF)(MUT)(L-2)(L-1)(R+1)(R+2)
  * arrow: AA[A>C]GT --> (L-2)(L-1)[(REF)>(ALT)](R+1)(R+2)
* REQUIRES a 2-bit human genome build

```
_,spectra_sbs = sa.spectra.get_spectra_from_maf(maf_df, reference='cosmic3_exome', hgfile='hg19.2bit')

print(spectra_sbs.head().iloc[:,:5])
```
**Arrow**
|         |   sample_551 |   sample_135 |   sample_118 |   sample_191 |   sample_124 |
|:--------|-------------:|-------------:|-------------:|-------------:|-------------:|
| AA[T>A]AA |            1 |            2 |            0 |            1 |            2 |
| AA[T>A]AC |            0 |            2 |            0 |            0 |            0 |
| AA[T>A]AG |            0 |            6 |            2 |            2 |            0 |
| AA[T>A]AT |            0 |            4 |            0 |            0 |            2 |
| ... |            1 |            2 |            0 |            1 |            0 |

_or_

**Word**
| context1536.word   |   sample_551 |   sample_135 |   sample_118 |   sample_191 |   sample_124 |
|:-----------------|-------------:|-------------:|-------------:|-------------:|-------------:|
| TAAAAA             |            1 |            2 |            0 |            1 |            2 |
| TAAAAC             |            0 |            2 |            0 |            0 |            0 |
| TAAAAG             |            0 |            6 |            2 |            2 |            0 |
| TAAAAT             |            0 |            4 |            0 |            0 |            2 |
| ...             |            1 |            2 |            0 |            1 |            0 |
---

#### PCAWG Composite spectra
* This encodes the 1536-base + 78-base + 83-base context
* REQUIRES a 2-bit human genome build
* Either SBS form should work  (Arrow vs Word)

```
_,spectra_id = sa.spectra.get_spectra_from_maf(maf_df, reference='pcawg_COMPOSITE', hgfile='hg19.2bit')

print(spectra_id.head().iloc[:,:5])
```

| context.pcawg   |   sample_0 |   sample_1 |   sample_2 |   sample_3 |   sample_4 |
|:-----------------|-----------:|-----------:|-----------:|-----------:|-----------:|
| AA[T>A]AA        |          0 |          1 |          3 |          1 |          0 |
| AA[T>A]AC        |          0 |          4 |          3 |          6 |          0 |
| AA[T>A]AG        |          2 |          2 |          2 |          9 |          0 |
| AA[T>A]AT        |          0 |          0 |          0 |          3 |          0 |
| ...              |          0 |          0 |          0 |          0 |          0 |
| AC>CA            |          0 |          0 |          0 |          3 |          0 |
| AC>CG            |          0 |          0 |          0 |          3 |          0 |
| ...              |          0 |          0 |          1 |          3 |          0 |
| Cdel1            |          0 |          1 |          3 |          1 |          0 |
| Cdel2            |          0 |          4 |          3 |          6 |          0 |
| Cdel3            |          2 |          2 |          2 |          9 |          0 |
| Cdel4            |          0 |          0 |          0 |          3 |          0 |
| ...              |          0 |          0 |          1 |          3 |          0 |

---

#### PCAWG Composite 96 spectra
* This encodes the 96-base + 78-base + 83-base context
* REQUIRES a 2-bit human genome build
* Either SBS form should work (Arrow vs Word)

```
_,spectra_id = sa.spectra.get_spectra_from_maf(maf_df, reference='pcawg_COMPOSITE96', hgfile='hg19.2bit')

print(spectra_id.head().iloc[:,:5])
```

| context.pcawg   |   sample_0 |   sample_1 |   sample_2 |   sample_3 |   sample_4 |
|:-----------------|-----------:|-----------:|-----------:|-----------:|-----------:|
| A[C>A]A          |          0 |          1 |          3 |          1 |          0 |
| A[C>A]C          |          0 |          4 |          3 |          6 |          0 |
| A[C>A]G          |          2 |          2 |          2 |          9 |          0 |
| A[C>A]T          |          0 |          0 |          0 |          3 |          0 |
| ...              |          0 |          0 |          0 |          0 |          0 |
| AC>CA            |          0 |          0 |          0 |          3 |          0 |
| AC>CG            |          0 |          0 |          0 |          3 |          0 |
| ...              |          0 |          0 |          1 |          3 |          0 |
| Cdel1            |          0 |          1 |          3 |          1 |          0 |
| Cdel2            |          0 |          4 |          3 |          6 |          0 |
| Cdel3            |          2 |          2 |          2 |          9 |          0 |
| Cdel4            |          0 |          0 |          0 |          3 |          0 |
| ...              |          0 |          0 |          1 |          3 |          0 |

---

#### PCAWG SBS + ID spectra
* This encodes the 1536-base + 83-base context
* REQUIRES a 2-bit human genome build
* Either SBS form should work (Arrow vs Word)

```
_,spectra_id = sa.spectra.get_spectra_from_maf(maf_df, reference='pcawg_SBS_ID', hgfile='hg19.2bit')

print(spectra_id.head().iloc[:,:5])
```

| context.pcawg   |   sample_0 |   sample_1 |   sample_2 |   sample_3 |   sample_4 |
|:-----------------|-----------:|-----------:|-----------:|-----------:|-----------:|
| AA[T>A]AA        |          0 |          1 |          3 |          1 |          0 |
| AA[T>A]AC        |          0 |          4 |          3 |          6 |          0 |
| AA[T>A]AG        |          2 |          2 |          2 |          9 |          0 |
| AA[T>A]AT        |          0 |          0 |          0 |          3 |          0 |
| ...              |          0 |          0 |          1 |          3 |          0 |
| Cdel1            |          0 |          1 |          3 |          1 |          0 |
| Cdel2            |          0 |          4 |          3 |          6 |          0 |
| Cdel3            |          2 |          2 |          2 |          9 |          0 |
| Cdel4            |          0 |          0 |          0 |          3 |          0 |
| ...              |          0 |          0 |          1 |          3 |          0 |

---

#### PCAWG 96 SBS + ID spectra
* This encodes the 96-base + 83-base context
* REQUIRES a 2-bit human genome build
* Either SBS form should work (Arrow vs Word)

```
_,spectra_id = sa.spectra.get_spectra_from_maf(maf_df, reference='pcawg_SBS96_ID', hgfile='hg19.2bit')

print(spectra_id.head().iloc[:,:5])
```

| context.pcawg   |   sample_0 |   sample_1 |   sample_2 |   sample_3 |   sample_4 |
|:-----------------|-----------:|-----------:|-----------:|-----------:|-----------:|
| A[C>A]A          |          0 |          1 |          3 |          1 |          0 |
| A[C>A]C          |          0 |          4 |          3 |          6 |          0 |
| A[C>A]G          |          2 |          2 |          2 |          9 |          0 |
| A[C>A]T          |          0 |          0 |          0 |          3 |          0 |
| ...              |          0 |          0 |          0 |          0 |          0 |
| Cdel1            |          0 |          1 |          3 |          1 |          0 |
| Cdel2            |          0 |          4 |          3 |          6 |          0 |
| Cdel3            |          2 |          2 |          2 |          9 |          0 |
| Cdel4            |          0 |          0 |          0 |          3 |          0 |
| ...              |          0 |          0 |          1 |          3 |          0 |
# Machine Learning for Malaria Transmission-Blocking Drug Screening

MSc dissertation project (Data Science & Analytics, University of Leeds) applying transfer-learning feature extraction and unsupervised clustering to fluorescence-microscopy images of malaria gametocytes, to detect visual "phenotype" changes caused by transmission-blocking drug candidates — reproducing and extending the PhIDDLI pipeline (Delves et al., 2023).

> **Headline finding:** verifying an inherited dataset-size figure (2,254 cells) directly against the source archive's manifest revealed it was the archive's entire 58-compound screen total, not the four-condition subset under study. The corrected, manifest-verified dataset (**264 cells**) is what every result below is built on.

---

## Key Results

| Metric | Value |
|---|---|
| Corrected dataset size | 264 cells (136 DMSO, 62 TC11, 39 TC39, 27 TC04) — down from an incorrectly reported 2,254 |
| Best configuration | ResNet50 → UMAP (nn=15, md=0.1) → k-means (k=3) |
| Best ARI (vs. true treatment) | **0.316** (permutation test, p < 0.001) |
| Class-balanced ARI | 0.244 |
| Feature extractor comparison | ResNet50 0.316 &nbsp;›&nbsp; DINOv2-small 0.254 &nbsp;›&nbsp; EfficientNet-B1 (original pipeline's own extractor) 0.157 |
| Headline biological finding | DMSO, TC04, and TC39 each form a distinct visual cluster; **TC11 never does** — consistent with its documented dual-target kinase mechanism (Crowther et al., 2016) |

---

## The Dataset Correction, in Brief

An earlier project milestone reported 2,254 cells across the four conditions studied here. Verifying that figure directly against the BioImage Archive's own manifest (rather than accepting it) proceeded in four steps:

1. Re-deriving cell counts independently from a subset of the same images gave ~1.7 cells/image, far below the ~27 cells/image the milestone figure implied.
2. The archive's live FTP file listing was tested directly and found unreliable at scale (inconsistent repeated results, missing files) — file discovery was moved to the archive's static manifest files instead.
3. The manifest revealed the archive already provides a pre-curated single-cell component (`Primary_Screen/Individual_cells`), distinct from the raw field-of-view images that had been self-cropped previously.
4. Cross-referencing that component against **all 58 compounds** in the full screen gave 2,244 cells — nearly identical to the incorrectly-attributed 2,254 — confirming the earlier figure was the archive's whole-screen total, not a number specific to this four-condition subset.

Full detail is in the dissertation write-up (Chapter 4) and reproduced step-by-step in Part 1 of the notebook.

---

## Pipeline Overview

1. **Cell identification** — pre-isolated single-cell images pulled directly from the BioImage Archive's own curated component (manifest-verified, not self-cropped).
2. **Feature extraction** — three pretrained extractors compared under an identical downstream pipeline: ResNet50, EfficientNet-B1 (the original PhIDDLI pipeline's own extractor), and DINOv2-small (self-supervised ViT).
3. **Dimensionality reduction** — PCA vs. UMAP vs. t-SNE, swept systematically.
4. **Clustering** — k-means vs. HDBSCAN, swept systematically (200 configurations per extractor, 600 total).
5. **Validation** — a permutation significance test, a class-balance sensitivity re-test, a semi-supervised UMAP experiment with proper held-out validation (checking for circularity), and a multi-seed stability check on the hierarchical/soft-clustering results.

See the notebook's own header for the full Part 1–10 breakdown.

---

## Repository Structure

```
malaria-phenotype-ml/
├── README.md
├── requirements.txt
├── LICENSE
├── notebooks/
│   └── malaria_pipeline.ipynb     # the full, self-contained pipeline (Colab-first)
├── data/                          # populated at runtime by Part 1 — not committed (see .gitignore)
└── figures/                       # populated at runtime by Part 9 — not committed (see .gitignore)
```

The notebook is intentionally single-file and self-contained ("run top to bottom, once, in order") — it was built and validated as a single Colab notebook, so it is kept that way here rather than split into modules, to avoid any risk of introducing inconsistencies with the results reported in the dissertation.

---

## Running It

**Google Colab (recommended — this is how it was built and run):**
1. Upload `notebooks/malaria_pipeline.ipynb` to [Google Colab](https://colab.research.google.com).
2. Runtime → Run all. Part 1 downloads the required data directly from the BioImage Archive over FTP — no API key needed, just a network connection.
3. Outputs (feature matrices, sweep results, figures) are written to `/content/data` and `/content/figures` inside the Colab session, and archived to zip files in Part 10.

**Locally:**
```bash
pip install -r requirements.txt
jupyter notebook notebooks/malaria_pipeline.ipynb
```
Update `DATA_ROOT` and `FIG_DIR` in Part 1 (currently `/content/data`, `/content/figures`) to local paths, and note that Part 2/3/last-cell feature extraction (TensorFlow + PyTorch/transformers) is significantly faster with a GPU, as in Colab's free tier.

---

## Data & Citation

Raw data is **not** included in this repository — it is downloaded at runtime directly from its source:

- **Dataset:** Delves, M.J. et al. (2023). *PhIDDLI Primary Screen and Timecourse Imaging Dataset.* BioImage Archive, accession [S-BIAD633](https://www.ebi.ac.uk/biostudies/bioimages/studies/S-BIAD633).
- **Original pipeline / methodology this work reproduces and extends:** Delves, M.J. et al. (2023). *Machine Learning-based Phenotypic Imaging to Characterise the Targetable Biology of Plasmodium falciparum Male Gametocytes for the Development of Transmission-Blocking Antimalarials.* bioRxiv. https://doi.org/10.1101/2023.04.22.537818

Please cite the original dataset/paper above if you build on the data itself; this repository's code is separately licensed as below.

**Note on the dissertation PDF:** the full written dissertation is not included in this repository — it contains a personal academic integrity declaration and student ID not intended for public posting. This repo covers the code only.

---

## License

MIT for the code in this repository. The underlying imaging dataset remains subject to the BioImage Archive's and original authors' own terms.

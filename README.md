# MR-EyeTrack

3D radial-spiral T1-weighted (pulseq LIBRE) MRI of the eye with simultaneous SR Research EyeLink
eye tracking. Each k-space readout (TR 8 ms) is labelled by gaze direction (up / down / left / right)
and each bin is reconstructed independently, giving four motion-resolved images per subject.

## Layout

| path | content |
| --- | --- |
| `sub-*/anat/*_acq-full_rec-regu_T1w` | LIBRE image from all fixation readouts (no gaze binning), regularised (STEVA) |
| `sub-*/anat/*_acq-full_rec-grid_T1w` | same readouts, gridding (MATHILDA) |
| `sub-*/anat/*_acq-mprage_T1w` | reference MPRAGE from the scanner (Siemens IDEA), converted from DICOM |
| `sourcedata/sub-*/rawdata` | Siemens twix `.dat`: `_T1wLIBRE` (main), `_HC` / `_BC` (head / body coil prescans), `_AdjCoilSens` |
| `sourcedata/sub-*/et` | eye tracking: original EyeLink / PsychoPy files (EDF, csv, log, psydat) and the BIDS conversion (`.tsv.gz` + `.json`) |
| `sourcedata/pulseq` | the `.seq` files defining the k-space trajectory (prescan and main) |
| `derivatives/mreyetrack-recon` | gaze-binned and other reconstructions, coil sensitivity maps |
| `derivatives/mreyetrack-binning` | ET masks and per-readout gaze labels, ET-MRI sync corrections |

## File naming

- `acq-up|down|left|right` gaze bin, `acq-full` all readouts.
- `rec-regu` regularised iterative compressed-sensing reconstruction (STEVA, 20 iterations, delta 1);
  `rec-grid` gridding (MATHILDA, the initial estimate of the iterative recon).
- `desc-keep50|keep25|keep5` random undersampling study: percentage of the clean readouts kept.
- Images are 240 x 240 x 240, about 1 mm isotropic, float32, RAS.

## Readout indexing

Of 82 368 acquired readouts (44 segments x 1872 shots), segment 1 of each shot (SI navigator) and the
first 14 shots (warm-up) are dropped, leaving 79 894 (43 x 1858). Labels use the same ordering.
In numpy: `mask.reshape(1872, 44)[14:, 1:].ravel()`; MATLAB is column-major.

## Authors

Jaime Barranco (1,2,3), Yiwei Jia (1,2,4), Eleonora Fornari (3,5), Oscar Esteban (1), Jean-Baptiste Ledoux (3,5),
Mauro Leidi (1,2), Helene Vitali (6,7), Cédric Campos Carvalho (8), Elena Najdenovska (8),
Laura Elena Raileanu (8), Jessica Bastiaansen (9,10), Micah M. Murray (2,3), Benedetta Franceschiello (1,2).

1. HES-SO University of Applied Sciences and Arts Western Switzerland.
2. The Sense Innovation and Research Centre, Lausanne and Sion, Switzerland.
3. Department of Radiology, Lausanne University Hospital and University of Lausanne, Lausanne, Switzerland.
4. Graduate School for Cellular and Biomedical Sciences, University of Bern, Bern, Switzerland.
5. CIBM Center for Biomedical Imaging, Lausanne, Switzerland.
6. Unit for visually impaired people, Italian Institute of Technology, Genova, Italy.
7. Department of Computer Science, Bioengineering, Robotics and Systems, University of Genoa, Genova, Italy.
8. School of Engineering and Management Vaud (HEIG-VD), HES-SO University of Applied Sciences and Arts Western
   Switzerland, Yverdon-les-Bains, Switzerland.
9. Department of Diagnostic, Interventional and Pediatric Radiology (DIPR), Inselspital, Bern University Hospital,
   University of Bern, Switzerland.
10. Translation Imaging Center (TIC), Swiss Institute for Translational and Entrepreneurial Medicine, Bern, Switzerland.

Lead contact: Benedetta Franceschiello (benedetta.franceschiello@hevs.ch).
Technical contact: Jaime Barranco (jaime.barrancohernandez@hevs.ch).
Code: https://github.com/MattechLab/MR-EyeTrack

## Ethics, funding and competing interests

The MR-Eye Track protocol was approved by the local ethics committee through the Swiss BASEC
(protocol number 2024-00165, 27 June 2024). The original protocol was approved by the cantonal ethics
committee (protocol number 2018-00240). Funding is listed in `dataset_description.json`.
The authors had a previous patent application related to the original study; it has expired, and the
authors declare no competing interests.

## Not included

The split k-space (mitosius, ~50 GB per subject) and the ROI-PCA compressed k-space are not shared; they
can be rebuilt from `sourcedata` + `derivatives/mreyetrack-binning` with the MR-EyeTrack pipeline.
The reference MPRAGE is not available for sub-010 and sub-011.

## Known issues

- Eye tracking of sub-007 and sub-009 started after the scan began (offsets -451 / -456 ms), unlike the
  other subjects (about +61 ms). All offsets, plus the per-subject EyeLink clock-rate error, are in
  `derivatives/mreyetrack-binning/sync_corrections.mat` and are already applied to the `_sync` labels.
- The `acq-full_rec-grid` images of sub-001 to sub-005 are stored without intensity normalization (values
  around 1e-8, against about 1-10 for every other image). They are intact, but a viewer with a fixed display
  window shows them black; rescale the display window to view them.

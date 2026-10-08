[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.nm000358-blue)](https://doi.org/10.82901/nemar.nm000358)

AJILE12: long-term naturalistic ECoG with wrist-movement events and behaviour labels (DANDI 000055)
====================================================================================================

Overview
--------
Electrocorticography (ECoG; grids, strips and in some participants depth electrodes) recorded
opportunistically from 12 participants during clinical long-term epilepsy monitoring at Harborview
Medical Center (Seattle, USA), with simultaneous video from which the authors estimated upper-body
pose, detected and visually validated wrist-movement initiation events, and annotated coarse
behavioural states (sleep/rest, TV, talking, eating, computer/phone, ...). One recording per
participant per monitoring day (days 3-7 after implantation), each covering one day from midnight
to midnight, 55 days in total. There is no experimental task.

This is an iEEG-BIDS representation of the ECoG released by the authors in NWB on DANDI:

  Peterson SM, Singh SH, Dichter B, Scheid M, Rao RPN, Brunton BW (2022). AJILE12: Long-term
  naturalistic human intracranial neural recordings and pose (Version 0.220127.0436). DANDI archive.
  https://doi.org/10.48324/dandi.000055/0.220127.0436  (license CC-BY-4.0)
  Data descriptor: Peterson SM et al. AJILE12: Long-term naturalistic human intracranial neural
  recordings and pose. Sci Data 9, 184 (2022). https://doi.org/10.1038/s41597-022-01280-y

Please cite both. The 30 Hz pose trajectories (9 keypoints, image pixels) are not converted; they
remain, unchanged, in the original NWB files under sourcedata/dandi-000055/ and on DANDI.

Cohort and acquisition
----------------------
From Peterson et al. (2022), Methods ("Participants", "Data collection"):
- 12 participants (8 male, 4 female; age 29.4 +/- 7.6 years, mean +/- SD) recorded during clinical
  epilepsy monitoring at Harborview Medical Center, Seattle, USA. ECoG electrodes were placed based on
  clinical need; participants were selected because they were generally active during monitoring and
  had ECoG electrodes near motor cortex.
- Semi-continuous ECoG and video were recorded passively during 24-hour clinical monitoring; monitoring
  lasted 7.4 +/- 2.2 days per participant, with 8.3 +/- 2.2 breaks per participant of 1.9 +/- 2.4 h.
  Only days 3-7 after implantation were included; days with corrupted or missing files were excluded.
- Acquisition rates: ECoG 1 kHz (released at 500 Hz after the authors' processing, see Signal), video 30
  frames per second. The paper does not name the amplifier, electrode manufacturer or acquisition
  reference; these fields are left unset.

Ethics
------
Peterson et al. (2022): all participants provided written informed consent; the protocol was
approved by the University of Washington Institutional Review Board (DANDI ethics record
STUDY00000623). This deposit redistributes the publicly released data under its CC-BY-4.0 license.

Contents
--------
12 participants (sub-01 ... sub-12), 55 recording days (ses-3 ... ses-7 = day after implantation),
64-126 ECoG/depth channels per participant at 500 Hz (plus EOGL/EOGR/ECGL/ECGR for sub-01), about
1295 h of recording including NaN monitoring breaks. 52 recordings span the full day (86400 s); 3 start
later in the day (sub-01 ses-7, sub-04 ses-3, sub-06 ses-3) and, like all others, end at midnight.
Events: 8088 wrist-movement events (ReachEvents, left or right wrist as stated per file) and 74339
coarse behaviour-label intervals.
(The data descriptor text reports 6931 visually validated wrist movement events; the ReachEvents
tables of the released NWB files, as converted here, hold 8088 rows.)

  ieeg/*_ieeg.vhdr/.vmrk/.eeg  BrainVision, IEEE float32, microvolts (resolution 1).
  ieeg/*_channels.tsv          channel type, author bad-channel flags and per-electrode source statistics.
  ieeg/*_electrodes.tsv        MNI coordinates from the source (see Coordinates).
  ieeg/*_events.tsv            wrist-movement events, coarse behaviour labels, NaN segments.
  sourcedata/dandi-000055/     byte-identical NWB files (+ dandiset.yaml); sourcedata_provenance.json
                               lists size, SHA-256 and DANDI asset id.

Signal
------
The .eeg payload is byte-for-byte the float32 array stored in acquisition/ElectricalSeries of each
NWB file (stored in microvolts; NWB conversion 1e-6 to volts), multiplexed, channel order as the
source electrode region. NaN samples (monitoring breaks) are kept as NaN. This conversion applied no
filtering, resampling, re-referencing or channel removal.

Processing already applied by the authors (Peterson et al. 2022, "ECoG data processing"): median
DC removal per electrode; data within 2 s of high-amplitude discontinuities set to 0; 1-200 Hz
band-pass; 60 Hz (and harmonics) notch; downsampling from 1 kHz to 500 Hz; re-referencing to the
common median of each grid, strip or depth electrode group. The NWB "filtering" column reads
"250 Hz lowpass" and is reported verbatim. "raw" in dataset_description therefore means "earliest
released form", not the acquisition signal.

Channel names: the source does not store the clinical contact labels. Names are generated as
<electrode group>_<NWB electrode id> (e.g. GRID_012); the number is not a clinical contact number.
Type is ECOG unless the group description states a depth electrode (then SEEG); the source group
descriptions are given in iEEGElectrodeGroups. status = bad where the source "good" column is False
(author QC: abnormal standard deviation or kurtosis). Source columns (SD, kurtosis, MAD, R2 of the
authors' decoding models) are kept in channels.tsv.

Clocks and events
-----------------
The NWB file holds two clocks. The ECoG starts at the NWB session_start_time (dates were replaced by
the authors; the time of day is kept) and has no stored timestamps. The pose, the coarse behaviour
labels (intervals/epochs) and the reach features (intervals/reaches, "Time of day (sec)") are on a
day clock that starts at midnight. The NWB ReachEvents timestamps are on the ECoG clock.
For each file this conversion computes D = UTC seconds-of-day of session_start_time and requires
  (a) D + ECoG duration = 86400 s within one sample (the ECoG ends at midnight), and
  (b) for every row, reaches.start_time - ReachEvents.timestamps - D is within 2 ms (the event
      resolution) of a whole number of days (0 except where the authors' time of day wrapped past
      midnight, e.g. 86440.332 s for an event 40.332 s after midnight).
All 55 recordings pass both gates (maximum residual 1 ms; one row in sub-08 ses-4 has a time of day
of 86440.332 s for an event 40.332 s after midnight). D is 0 for 52 recordings and 30682.414,
30048.322 and 29240.566 s for the 3 recordings that start later in the day.
Event onsets (seconds from the first ECoG sample):
  reach_onset   ReachEvents timestamp, with the reach features of the same row and its duration;
  coarse_label  epochs start - D; labels that start before the first ECoG sample have negative onsets
                (validator warning SUSPICIOUS_NEGATIVE_EVENT_ONSET; sample is n/a for them);
  nan_segment   runs of samples that are NaN on every channel in the source.
source_time_of_day keeps the original day-clock time. Coarse-label boundaries follow the source
converter, which assigns each change to the last frame of the previous label (one 30 Hz frame).

Coordinates
-----------
x/y/z are copied from the NWB electrodes table: electrodes were localised with FieldTrip
(pre-operative MRI co-registered to post-operative CT, manual selection) and warped to MNI space
(Peterson et al. 2022). The MNI template variant is not stated, so coordsystem.json uses "Other"
with this description, units mm.

Participants and privacy
------------------------
participants.tsv gives, per participant: species (NWB subject record) and, from Peterson et al. (2022)
Table 2 "Individual participant characteristics", age (years), sex (M/F), hemisphere_implanted (L/R),
recording_days_used, surface_electrodes_good / surface_electrodes_total and depth_electrodes_good /
depth_electrodes_total; participants.json describes each column and its source. Paper participants
P01-P12 = sub-01 ... sub-12 (the electrode totals equal the electrodes.tsv row counts and the recording
days equal the number of ses-* folders). The authors stripped recording dates (session dates in the
NWB are placeholders, 2000-01-0x); BIDS files carry no dates.

How to load
-----------
Example with MNE-Python / MNE-BIDS (fetch the files you need first, e.g. `nemar dataset get` or
`datalad get`):

    from mne_bids import BIDSPath, read_raw_bids
    bp = BIDSPath(root="<dataset root>", subject="01", session="3", task="naturalistic",
                  datatype="ieeg", suffix="ieeg", extension=".vhdr")
    raw = read_raw_bids(bp)        # 500 Hz, microvolts in file, volts in MNE; NaN = monitoring break
    import pandas as pd
    ev = pd.read_csv(bp.copy().update(suffix="events", extension=".tsv").fpath, sep="\t")
    reaches = ev[ev.trial_type == "reach_onset"]

Each file spans a full day (up to 86400 s at 500 Hz, about 43 M samples per channel); load lazily or
crop before calling load_data().

Conversion checks
-----------------
- Source: 55/55 DANDI assets (845,869,698,341 bytes) verified by SHA-256 against the DANDI digests;
  sourcedata copies re-verified after copying.
- Signal: every .eeg payload is bit-identical (incl. NaN) to the NWB stored float32 values in source
  channel order; MNE reads the same channel names, rate and sample count, and its scaled values match
  stored x 1e-6 V on start/middle/end windows.
- Events: reach_onset onsets equal the NWB ReachEvents timestamps; coarse_label onsets equal the NWB
  epochs start - D (1e-6 s); clock gates as above.
- BIDS validator (bids-validator 3.0.2): 0 errors; warnings are recommended fields not documented by
  the source and SUSPICIOUS_NEGATIVE_EVENT_ONSET for labels that begin before the ECoG start.

Provenance
----------
Source: DANDI:000055 version 0.220127.0436, all 55 assets downloaded with the dandi CLI (0.81.0)
on 2026-10-06 and verified against the DANDI SHA-256 digests. Conversion scripts:
laneB_ieeg002_bids.py, checks laneB_ieeg002_check.py (h5py 3.16, numpy 2.5, MNE 1.13). An earlier
automated nwb2bids attempt for this Dandiset (github.com/bids-dandisets/000055) converted 0 of the
sessions; no BIDS copy of these recordings was found on DANDI, OpenNeuro or NEMAR as of 2026-10-06.

## Atlas labels of the electrode positions (added 2026-10-08)

Each `electrodes.tsv` that has coordinates now has two derived columns, `atlas_label_AAL3v1` and `atlas_label_DesikanKilliany`. They are an atlas lookup of the coordinates already in the file (MNI coordinates used as given), made for NEMAR; they are not labels given by the authors, and the coordinates themselves are unchanged. Method and caveats: `electrodes.json`. 4787 of 5432 contacts received a label.

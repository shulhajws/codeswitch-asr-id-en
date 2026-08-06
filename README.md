# Indonesian–English Code-Switching Speech Recognition Dataset

Data, transcriptions, and annotations from the undergraduate final project
*"Handling Code-Switching in Automatic Speech Recognition for Low-Resource Language Pairs: An Indonesian–English Case Study"*, School of Electrical Engineering and Informatics, Institut Teknologi Bandung.

This repository accompanies a study on adapting Whisper for intra-sentential Indonesian–English code-switching under limited labelled data. It contains the code-switching text corpora, the manually transcribed evaluation set, the pseudo-labelled training transcriptions, and the metadata needed to reconstruct all audio.

---

## What is and is not included

**Included.** All text data produced in this work: synthetic code-switching text corpora, manual reference transcriptions with word-level language annotation, pseudo-label transcriptions with their filtering scores, corpus statistics, and manifests linking every audio segment to its source video and timestamps.

**Not included.** Audio derived from YouTube. The training and evaluation audio comes from publicly available YouTube videos whose copyright belongs to the respective channel owners. Redistributing that audio would violate both the YouTube Terms of Service and the owners' rights. Instead, this repository provides manifests containing video IDs, URLs, and timestamps so the audio can be reconstructed locally. See [Reconstructing the audio](#reconstructing-the-audio).

---

## Repository structure

```
.
├── LICENSE
├── CITATION.cff
├── README.md
├── text_corpus/
│   ├── indo_eng_codeswitch_text_20000_tagged.csv
│   ├── indo_eng_codeswitch_text_20000_tagged_phonetic.csv
│   ├── indo_eng_codeswitch_text_10000_tagged.csv
│   └── dataset_codeswitching_1_5000_filtered_3929.csv
├── test_set/
│   ├── test_set_manifest.csv
│   ├── data_uji_ground_truth.csv
│   └── data_uji_ground_truth_with_word_tag.csv
├── pseudo_labels/
│   ├── chunk_to_video.csv
│   ├── pseudo_labels_for_training.csv
│   ├── pseudo_labels_filtered.csv
│   └── pseudo_labels_rejected.csv
└── metadata/
    ├── daftar_video_data_latih.csv
    ├── statistik_korpus.csv
    └── oov_data_uji.csv
```

---

## Dataset overview

| Component | Size |
|---|---|
| Evaluation set (manually transcribed) | 98 utterances, 39.7 minutes, 5,625 words |
| Training audio (pseudo-labelled) | 4,416 segments, ≈29.7 hours, 295,852 words |
| Source videos, training | 69 YouTube videos, 65 distinct speakers |
| Source videos, evaluation | 2 YouTube videos, 3 speakers |
| Synthetic text corpora | 20,219 / 10,000 / 3,929 sentences |

Code-switching density is comparable across splits: 32.4% of training tokens and 35.4% of evaluation tokens are English insertions in an Indonesian matrix.

---

## File descriptions

### `text_corpus/`

Synthetic Indonesian–English code-switching sentences generated for this study, used to produce synthetic speech via text-to-speech. Language identity is marked inline with `<|id|>` and `<|en|>` span tags.

| Column | Description |
|---|---|
| `id` | Sentence identifier |
| `text` | Sentence with inline `<|id|>` / `<|en|>` language span tags |

`indo_eng_codeswitch_text_20000_tagged_phonetic.csv` additionally contains `original_tagged` (the tagged sentence), `ground_truth` (the target transcription), and `tts_input` (the sentence with English words respelled phonetically for the TTS front-end).

### `test_set/`

**`test_set_manifest.csv`** — the primary evaluation manifest.

| Column | Description |
|---|---|
| `file_name` | Segment file name (`test_001.wav` … `test_098.wav`) |
| `video_id`, `video_url` | Source YouTube video |
| `start_time`, `end_time` | Segment boundaries as `MM:SS` |
| `start_sec`, `end_sec`, `duration_sec` | Segment boundaries in seconds |
| `ground_truth` | Manual reference transcription |
| `n_words`, `n_words_id`, `n_words_cs` | Word counts: total, Indonesian, English (code-switched) |

**`data_uji_ground_truth_with_word_tag.csv`** — word-level language annotation, produced manually.

| Column | Description |
|---|---|
| `ID` | Zero-based utterance index (`ID` = *n* corresponds to `test_{n+1:03d}.wav`) |
| `Ground_Truth` | Reference transcription |
| `Indo_Words`, `CS_English_Words` | Comma-separated word lists per language |
| `Indo_Count`, `CS_Count`, `GT_Word_Count` | Corresponding counts |

This annotation is what makes code-switch-focused evaluation possible: it identifies exactly which reference tokens are English insertions, so errors can be scored on those positions specifically rather than diluted across the dominant matrix language.

### `pseudo_labels/`

Training transcriptions produced by transcribing YouTube audio with Whisper-large-v3 and validating each segment against a second model, keeping only segments where both models agree.

**`chunk_to_video.csv`** — links every training segment to its source video.

| Column | Description |
|---|---|
| `file_name` | Segment file name (`00001.wav` …) |
| `source_video_title` | Source video title as downloaded |
| `video_id`, `video_url` | Source YouTube video, empty when unavailable |
| `pseudo_label` | Transcription used for training |

**`pseudo_labels_filtered.csv`** and **`pseudo_labels_rejected.csv`** — all 7,069 candidate segments with their filtering scores, including the segments that were rejected. Three agreement criteria were applied between the two transcriptions: word error rate agreement (`gate1_pass`), code-switch token overlap measured by Jaccard index (`gate2_pass`), and length ratio plausibility (`gate3_pass`). 4,416 segments (62.5%) passed all three.

Rejected segments are published deliberately: they allow the filtering thresholds to be re-examined or replaced without re-running the transcription step.

### `metadata/`

| File | Contents |
|---|---|
| `daftar_video_data_latih.csv` | All 69 source videos with URL, segment count, and estimated duration |
| `statistik_korpus.csv` | Language composition of every data source |
| `oov_data_uji.csv` | The 76 evaluation word types absent from all training data |

---

## Reconstructing the audio

All audio can be regenerated from the manifests. The general procedure:

1. Read `video_url` and the timestamp columns from `test_set_manifest.csv` (evaluation) or `chunk_to_video.csv` (training).
2. Download each video's audio track with a tool such as `yt-dlp`.
3. Resample to 16 kHz mono.
4. Slice according to the timestamps.

Training segments are uniform 30-second windows cut sequentially from each video, so segment *n* of a given video begins at (*n* − 1) × 30 seconds.

Note that this reconstructs audio only. Users are responsible for complying with the YouTube Terms of Service and applicable copyright law.

---

## Synthetic speech

Synthetic audio used in part of this study was generated with [Chatterbox](https://github.com/resemble-ai/chatterbox) (Resemble AI, MIT licence) from the sentences in `text_corpus/`. Generation is deterministic given the same model, voice reference, and configuration, so the audio can be regenerated from the text corpora rather than downloaded.

Chatterbox embeds an imperceptible PerTh neural watermark in every generated clip. This does not affect the audio's usability for speech recognition experiments, but it does mean the synthetic audio remains identifiable as synthetic.

---

## Known limitations

**One source video is no longer available.** One of the 69 training videos has been removed or made private since collection, affecting 75 of 4,416 segments (1.7%). Its transcriptions remain in the release; only its audio cannot be reconstructed.

**41 segments lack a source mapping.** These segments have transcriptions but no recoverable link to a source video, and appear with an empty `video_id`.

**Four evaluation timestamps need verification.** Segments `test_037`, `test_044`, `test_075`, and `test_076` were transcribed in two passes whose text differs slightly; the timestamps for these four should be checked against the source video before use.

**Speaker overlap between splits.** Two of the three evaluation speakers also appear in a small portion of the training data, at 0.68% and 2.06% of training segments respectively. The third speaker does not appear in training at all. Evaluation audio itself never overlaps with training audio: the two evaluation videos are excluded from training entirely.

**Pseudo-labels are not manual transcriptions.** Training transcriptions are model-generated and filtered by inter-model agreement, not verified by a human. The evaluation set is manually transcribed and is the only fully reliable reference in this release.

---

## Licence

Data, transcriptions, annotations, and documentation are released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

This licence does **not** extend to the YouTube content referenced by the manifests. Copyright in that content remains with the respective channel owners, and this repository does not redistribute it. See `LICENSE` for the full terms in Indonesian and English.

---

## Citation

If you use this data, please cite this repository. See `CITATION.cff`, or use the DOI issued for the archived release.

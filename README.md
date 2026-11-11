# BoJ Tone Tracker

**[Live demo on Hugging Face Spaces](https://huggingface.co/spaces/Kukai-7/boj-tone-tracker)**

Paste a sentence from a Bank of Japan (BoJ) monetary policy statement and a fine-tuned language model tells you whether its wording leans **hawkish** (tighter policy), **dovish** (easier policy) or **neutral**. A second tab scores every English *Statement on Monetary Policy* since 2013 and plots the resulting tone index against the Nikkei 225.

Everything runs in your browser: no server, no API key.

*Un petit outil pour lire le ton de la Banque du Japon, phrase par phrase.*

## Why

Central bank wording moves markets before rates do. Turning statements into a time series is a standard NLP task in finance, and the BoJ is a good test case: a decade at the zero lower bound, negative rates and yield curve control, then normalisation from 2024. This Space is a compact, end-to-end example of that workflow: data collection, a pre-trained transformer, aggregation into an index, and comparison with a market series.

## Method

1. **Corpus.** The English *Statements on Monetary Policy* and other main decision releases on [boj.or.jp](https://www.boj.or.jp/en/mopo/mpmdeci/state_all/index.htm), January 2013 to September 2026: 133 documents, 126 meetings, 2,591 sentences after cleaning. Reference annexes, attendance lists, voting records, the 2016 background assessment, the 2024 policy review and joint statements with the government are excluded. Every meeting keeps its source URL (`data/timeline.json`).
2. **Model.** [`mrince/CBRT-RoBERTa-HawkishDovish-Classifier`](https://huggingface.co/mrince/CBRT-RoBERTa-HawkishDovish-Classifier), a RoBERTa-base model fine-tuned on 7,200 sentences from the Central Bank of the Republic of Türkiye's policy committee summaries (MIT licence). Its card reports 87% accuracy on its own test set, with labels 0 = neutral, 1 = hawkish, 2 = dovish.
3. **In-browser inference.** `model.onnx` is a hand-built ONNX graph of the RoBERTa classifier (about 60 KB) whose weights are read, unchanged, from the original `model.safetensors` of the model repository via ONNX external data. Nothing is copied or re-hosted. The graph was checked against the PyTorch implementation (max absolute difference 1e-7), and on real weights it reproduces the example given in the model card. Runtime: ONNX Runtime Web, on the GPU through WebGPU when the browser offers it, otherwise on the CPU (WebAssembly); sentences are scored in batches of eight. Tokenizer: Transformers.js.
4. **Sentence view.** Each sentence gets a label and class probabilities. The score shown is the average of P(hawkish) minus P(dovish), from -1 to +1.
5. **Tone index.** For each meeting: (hawkish sentences - dovish sentences) / sentences, a count-based measure in the spirit of Shah, Paturi and Chava (2023, *Trillion Dollar Words*). A 4-meeting rolling mean smooths the series.
6. **Market overlay.** Nikkei 225 month-end close (Yahoo Finance), taken from my [nikkei-225-drivers](https://github.com/Kukai-7/nikkei-225-drivers) project.

## What it finds

Of 2,591 sentences, 62% are neutral, 21% dovish and 16% hawkish. The index sits below zero through the easing years (2016 and 2020 to 2021 are the most dovish) and turns clearly positive in 2026 as the policy rate rises. Single meetings are noisy; the 4-meeting average is the line to read.

## Limitations

- **Not trained on the BoJ.** The model learned from Turkish central bank summaries. BoJ language (yield curve control, ETF purchases, "the Bank will not hesitate...") differs, so expect errors. A BoJ-specific model exists (Georgia Tech's *World of Central Banks* series) but is gated; it would be a natural upgrade.
- **A reading of words, not of policy.** The model sees wording only. It ignores the level of rates, the yield curve target and market expectations, and cannot tell a routine sentence from a market-moving one.
- **Level, not change.** The March 2024 exit from negative rates scores slightly dovish (-0.10), because the statement also stresses that accommodative conditions will be maintained. Markets read the change; the model reads the words.
- **Inflation talk reads as hawkish.** Price projections ("the CPI will be pushed up") are often labelled hawkish, even inside easing announcements such as January 2013.
- **Easing continuity reads as dovish.** Sentences like "the Bank will continue with QQE" are labelled dovish, which is right in level but says nothing about *change*. Most of 2013 to 2023 therefore sits below zero.
- **Short recent statements.** Since 2024 the BoJ statement is one to three sentences (the assessment moved to the Outlook Report), so single meetings swing a lot.
- **Overconfidence.** Probabilities are often close to 0 or 1; they are not calibrated measures of certainty.
- **No causality.** Two series side by side say nothing about cause; the yen and global equities explain far more of the Nikkei.
- **Text extraction noise.** Sentences from PDFs can carry line-break or footnote artefacts.
- **Heavy first load.** Scoring your own text needs about 500 MB of weights, downloaded once and then kept on the device (Cache Storage). The tone chart and the example sentences need no download at all.

## Files

| file | what |
|---|---|
| `index.html` | the app |
| `model.onnx` | graph only, weights streamed from the model repo |
| `data/timeline.json` | meeting-level counts, tone, source URL, and the Nikkei series used by the chart |

## Run locally

The app is static. Serve the folder with any web server, for example `python -m http.server`, and open `index.html`. The weights are fetched from the model repository on first use.

## Related work

- [nikkei-225-drivers](https://github.com/Kukai-7/nikkei-225-drivers): OLS econometrics and Japanese Fama-French factors on the Nikkei 225.
- [Ming Crisis Atlas](https://huggingface.co/datasets/Kukai-7/ming-crisis-atlas): historical disaster records of Ming China, 1368 to 1644.
- [GitHub: Kukai-7](https://github.com/Kukai-7)

## Credits and licences

- Statements: © Bank of Japan, reproduced for non-commercial purposes with the source credited, as its [copyright policy](https://www.boj.or.jp/en/about/copyright.htm) allows.
- Model: Muhammet Rıdvan İnce, MIT.
- Code: MIT.

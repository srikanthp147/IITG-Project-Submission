# ZentroMart VoiceOps Support Agent 

## Overview
A customer-support assistant for the supplied synthetic ZentroMart capstone dataset. The implementation includes policy-aware retrieval (Sentence Transformer embeddings with FAISS Retrieval technique), source authority weighting, product-catalog lookup, synthetic order verification, ticket creation, conversation replay, phrase-based evaluation, and optional Gnani speech input/output.

## Dataset and reference date
The supplied dataset is included as `Capstone_Project_Dataset_FTTT_2026.zip` and extracted to `Capstone_Project_Dataset_FTTT_2026/`. Use the dataset reference date **20 October 2026** for date calculations. All dataset records are synthetic.

# Github URL
https://github.com/srikanthp147/IITG-Project-Submission

## Setup
1. Use Python 3.10+ (Python 3.11 is a practical default).
2. Create a virtual environment: `python -m venv .venv` and activate it.
3. Install dependencies: `pip install -r requirements.txt`.
4. If the selected Hugging Face model requires authorization, set `HF_TOKEN` in the environment or use a model to which your account has access. Do not place tokens in source files.
5. For voice, set `GNANI_API_KEY` as an environment secret. Voice is optional for text evaluation.

## Run the complete evaluation
`python run.py`

The runner executes deterministic unit tests, loads `google/gemma-2-2b-it`, audits the KB/retriever, evaluates all 52 practice questions in prompt-only and RAG modes, replays all 23 conversations (96 customer turns), writes comparison and failure-review files, and writes the report under `outputs/`. It attempts optional audio processing only if `GNANI_API_KEY` is set and audio files are placed in `sample_inputs/audio/`.

Useful options:
- `python run.py --skip-voice` — run text evaluation only.
- `python run.py --skip-evaluation` — load model and run unit tests/smoke checks without evaluating all questions.

A model-backed run needs the listed dependencies, model download/access, and sufficient RAM/GPU memory. If model loading fails, the runner reports the reason; do not treat previous result CSVs as a new run.

## Evaluation metric and existing outputs
`simple_pass` is a phrase-based check of `must_include` and `must_not_include`; it is not semantic accuracy. The `outputs/previous_run/` files are the evaluation snapshot supplied during project assistance, not a rerun performed while assembling this ZIP.The model-backed evaluation could not be rerun in the packaging environment because Transformers and the selected model weights were not installed/available. Run `python run.py` in the configured environment to regenerate current outputs before claiming these as final-run results.

## Voice testing
The dataset supplies no prerecorded audio. Record a short test utterance and place the audio file in `sample_inputs/audio/`. With a valid Gnani key and installed client, the runner writes transcript, text reply, generated WAV, source JSON, and a manifest to `outputs/voice_test_outputs/`. Check the provider's current API/library instructions if method signatures have changed.

## Privacy and security
Never commit `.env`, API keys, Hugging Face tokens, or actual credentials. The included `.env.example` intentionally contains blank secrets. The order-verification flow is a synthetic capstone simulation, not production authentication.
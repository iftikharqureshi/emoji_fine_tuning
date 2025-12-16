# Emoji-Style Chatbot Fine-Tuning

Guide for reviewers: what lives here, how the fine-tuning notebook works, and how to reproduce a quick run.

## Overview
- Objective: fine-tune `gpt-3.5-turbo` so the assistant replies only with parenthesized emoji labels (e.g., `(party)`, `(sleeping)`).
- Workflow is captured in `emoji_fine_tuning.ipynb`: upload the training set, launch a fine-tuning job, monitor it, inspect events, and smoke-test the resulting model.
- Training data sits in `data/emoji_ft_train.jsonl` and uses the chat format required by the OpenAI API.

## Repo Contents
- `emoji_fine_tuning.ipynb` — end-to-end notebook for uploading data, creating a job, polling status, viewing events, and testing the tuned model.
- `data/emoji_ft_train.jsonl` — 569 chat examples (349 unique emoji labels); each line is a `messages` array with a fixed system prompt plus user/assistant pairs.

## Data Notes
- Format per line:
  ```json
  {"messages": [
    {"role": "system", "content": "You're a chatbot that only responds with emojis!"},
    {"role": "user", "content": "I just passed my driving test!"},
    {"role": "assistant", "content": "(party)"}
  ]}
  ```
- Assistant targets are text labels wrapped in parentheses rather than Unicode emoji. The notebook’s final analysis cell calls this out and notes label frequency (top labels like `(upsidedownface)` and `(devil)` appear 7×).
- There is no separate validation file in the repo; the notebook fine-tunes against this single training split.

## Running the Notebook
1) Install dependencies (Python 3.9+): `pip install openai jupyter` (add `python-dotenv` if you prefer env files).  
2) Set your API key in the environment: `export OPENAI_API_KEY=...`. The notebook will prompt via `getpass` if it’s missing.  
3) Open `emoji_fine_tuning.ipynb` in Jupyter/VS Code and run the cells in order. Key parameters are set up front: `TRAINING_FILE_PATH`, `BASE_MODEL` (`gpt-3.5-turbo`), `MODEL_SUFFIX` (`emoji-v1`), and `N_EPOCHS` (3).  
4) The job monitoring cell polls until the status reaches `succeeded/failed/cancelled`, then prints the `fine_tuned_model` id. The final cell runs a quick chat completion to sanity-check the output style.

## What to Review
- Confirm the training JSONL adheres to the chat fine-tuning schema and that assistant targets match the intended emoji-label style.
- Check that the notebook guards secrets (uses `getpass` and `OPENAI_API_KEY`) and that the polling logic handles terminal job states.
- Validate that the test prompt in the final cell reflects the desired behavior and cost profile before running (fine-tuning incurs API charges).

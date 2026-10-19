# Packet and System Log Analysis

Review code for Llama fine-tuning and a two-agent packet and system log analysis
system. Download the ZIP from this anonymous repository and extract it before
running the commands below.

## 1. Llama fine-tuning

The training code prepares packet and log examples and fine-tunes Llama 3 8B
with LoRA. Python 3.11, Poetry, and an NVIDIA GPU with CUDA are required.

```bash
cd llama-pcaplog
poetry env use python3.11
poetry install
poetry run python -m pip install peft bitsandbytes datasets
poetry run python -c "from huggingface_hub import login; login()"
poetry run python model/training/train_model.py \
  --model-name meta-llama/Meta-Llama-3-8B \
  --dataset-dir model/datasets \
  --output-dir outputs \
  --hf-model-name your-hf-username/Llama-PcapLog
```

Replace `your-hf-username` with your own account. The script saves the model and
uploads the merged weights to that account. Large JSON training files are split
into smaller JSON arrays for the review service. All records are retained and
the existing loader reads every JSON file in the dataset directory.

## 2. Model evaluation

Evaluation covers basic analysis, advanced analysis, vulnerability classification,
and code generation. The fine-tuned model's download address is replaced with
the local path `./models/Llama-PcapLog` in this review copy. Put the trained model
and tokenizer there, or change the path in `evaluation/run_benchmark.py`.

Run from `llama-pcaplog/`:

```bash
poetry run python evaluation/run_benchmark.py \
  --benchmark_dir evaluation \
  --output_dir evaluation/results
```

The runner also evaluates the comparison models listed in `model_configs`.
Setting `OPENAI_API_KEY` additionally enables the GPT-4o comparison. Questions
are in `evaluation/test/test_dataset.json`. Code generation tasks are in the
evaluation script.

## 3. Two-agent system

The packet agent retrieves packet evidence with `tshark`. The log agent retrieves
system log evidence with `grep`. Each agent interprets its evidence with a local
model. A coordinator collects their reports and combines their verdicts.

From the repository root, install the system dependencies:

```bash
uv sync
cp .env.example .env
ollama list
```

Register the trained model in Ollama separately. In `.env`, use its actual name
from `ollama list` and set the local connection as follows:

```dotenv
OPENAI_API_KEY=
GOOGLE_API_KEY=
OLLAMA_HOST=http://localhost:11434
MAS_LLM_BACKEND=ollama
MAS_LLM_MODEL=Llama-PcapLog-tool:latest
REDIS_HOST=
REDIS_PORT=6379
PUBSUB_LOG_PATH=results/pubsub_messages.jsonl
```

Start Ollama with `ollama serve` in another terminal if it is not already running.
Local evaluation uses the in-memory message bus and does not require Redis.

## 4. System evaluation

Prepare the packet file and corresponding log file:

```text
data/CIC-IDS2017/Tuesday-WorkingHours.pcap
data/CIC-IDS2017/syslog_threats.log
```

These raw runtime inputs are not bundled. If the log file is absent, the code
uses `data/CIC-IDS2017/Tuesday-WorkingHours_converted.csv` instead. Evaluation
questions and expected verdicts are in `results/qca_test_dataset_100.json`.

```bash
./run.sh experiment
```

This compares the multi-agent system, single-agent analysis, and the rule-based
baseline over three runs. Outputs are saved in `results/` as
`final_experiment_results.csv`, `full_metrics_report.csv`, and
`final_statistical_metrics.csv`. Agent messages are saved as
`pubsub_messages.jsonl`.

The multi-agent evaluation uses the Ollama settings. The single-agent comparison
uses `OPENAI_API_KEY`. Missing or failed model calls use a rule-based fallback.
Check the execution logs before treating an execution as an LLM evaluation.

The original web application and its run command remain in the source. It uses
an automatic OpenAI/fallback path rather than `MAS_LLM_MODEL`. Videos, figures,
personal paths, and machine-specific environment files are omitted from this
review copy.

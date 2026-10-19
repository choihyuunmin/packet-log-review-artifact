# Model Training and Evaluation

See the [review artifact instructions](../README.md) for setup and commands.

- `model/processors/`: packet and system log preprocessing.
- `model/datasets/`: training examples in JSON arrays.
- `model/training/train_model.py`: Llama fine-tuning with LoRA.
- `evaluation/run_benchmark.py`: model evaluation.
- `evaluation/test/`: evaluation questions and expected responses.
- `evaluation/benchmark_results/`: included model evaluation results.
- `web/`: model web application source.

Run model commands from this directory. Use a local model path for the fine-tuned
weights. Author-specific model download addresses are omitted from the review copy.

# Neural Magic MLPerf Inference Benchmarks for Natural Language Processing

This is the reference implementation for MLPerf Inference benchmarks for Natural Language Processing.

The chosen model is BERT-Large performing SQuAD v1.1 question answering task.

## Supported Models

| model | framework | accuracy | dataset | model link | precision | notes |
| ----- | --------- | -------- | ------- | ---------- | --------- | ----- |
| BERT-Large | ONNX | f1_score=90.6% | SQuAD v1.1 validation set | ["zoo:nlp/question_answering/bert-large/pytorch/huggingface/squad/base-none"](https://sparsezoo.neuralmagic.com/models/nlp%2Fquestion_answering%2Fbert-large%2Fpytorch%2Fhuggingface%2Fsquad%2Fbase-none) | fp32 | This model is the result of fine-tuning the BERT large uncased model on the SQuAD v1.1 datasets. It achieves 90.6% accuracy on the validation dataset. See the included recipe for training instructions. |
| BERT-Large | ONNX | f1_score=90.3% | SQuAD v1.1 validation set | ["zoo:nlp/question_answering/bert-large/pytorch/huggingface/squad/pruned80_quant-none-vnni"](https://sparsezoo.neuralmagic.com/models/nlp%2Fquestion_answering%2Fbert-large%2Fpytorch%2Fhuggingface%2Fsquad%2Fpruned80_quant-none-vnni) | Fine-tuned based on the PyTorch model and converted with [bert_tf_to_pytorch.py](bert_tf_to_pytorch.py) | int8 | This model is the result of sparse transferring the 80% pruned BERT large uncased model, created using Prune OFA method described in Prune Once for All: Sparse Pre-Trained Language Models, to the SQuAD v1.1 datasets, then quantizing the resulting sparse model. It achieves 90.3% F1 on the validation dataset. See the included recipe for training instructions. |

## Disclaimer
This benchmark app is a reference implementation that is not meant to be the fastest implementation possible.

## Commands

Please run the following commands:

- `make setup`: initialize submodule, download datasets, and download models.
- `make build_docker`: build docker image.
- `make launch_docker`: launch docker container with an interaction session.
- `python3 run.py --backend=deepsparse --batch_size=1 --model_path="zoo:nlp/question_answering/bert-large/pytorch/huggingface/squad/pruned80_quant-none-vnni" --scenario=[Offline|SingleStream|MultiStream|Server] [--accuracy]`: run the harness inside the docker container. Performance or Accuracy results will be printed in console.

## Details

- SUT implementation is in [deepsparse_SUT.py](tf_SUT.py). QSL implementation is in [squad_QSL.py](squad_QSL.py).
- The script [accuracy-squad.py](accuracy-squad.py) parses LoadGen accuracy log, post-processes it, and computes the accuracy.
- Tokenization and detokenization (post-processing) are not included in the timed path.
- The inputs to the SUT are `input_ids`, `input_make`, and `segment_ids`. The output from SUT is `start_logits` and `end_logits` concatenated together.
- `max_seq_length` is 384.

## License

Apache License 2.0

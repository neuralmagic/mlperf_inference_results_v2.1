# Neural Magic's DeepSparse MLPerf Submission

This is the repository of Neural Magic's DeepSparse submission for [MLPerf Inference Benchmark v2.1](https://www.mlperf.org/inference-overview/).

## Benchmark

Our MLPerf Inference v2.1 submission contains the following results for the BERT-Large SQuAD v1.1 question answering task:

| Benchmark            | % of BERT-Large FP32 Accuracy (F1=90.874) | Scenarios             |
|:--------------------:|:-----------------------------------------:|:---------------------:|
| BERT-Large Prune OFA | 99.48% (F1=90.41)                         | SingleStream, Offline |
| [oBERT-Large](obert_large.md)          | 99.27% (F1=90.21)                         | SingleStream, Offline |
| oBERT-MobileBERT     | 99.39% (F1=90.32)                         | SingleStream, Offline |

The benchmark implementation and models are stored in the [code/bert/deepsparse](code/bert/deepsparse) directory which contains a `README.md` detailing instructions on how to set up the benchmark. An example of the commands used to generate this submission is stored in [submission.sh](submission.sh).

The benchmark was evaluated using a server with two Intel(R) Xeon(R) Platinum 8380 (IceLake) CPUs with 40 cores each.

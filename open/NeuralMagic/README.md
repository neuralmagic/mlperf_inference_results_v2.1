# Neural Magic's DeepSparse MLPerf Submission

This is the repository of Neural Magic's DeepSparse submission for [MLPerf Inference Benchmark v2.1](https://www.mlperf.org/inference-overview/).

## Benchmark

Our MLPerf Inference v2.1 submission contains the following results for the BERT-Large SQuAD v1.1 question answering task:

| Benchmark            | % of BERT-Large FP32 Accuracy | Scenarios             |
|:--------------------:|:-----------------------------:|:---------------------:|
| BERT-Large Prune OFA | 99.36%                        | SingleStream, Offline |
| oBERT-Large          | 99.28%                        | SingleStream, Offline |
| oBERT-MobileBERT     | 99.28%                        | SingleStream, Offline |

The benchmark is stored in the [code/bert/](code/bert) directory which contains a `README.md` detailing instructions on how to set up the benchmark. 
The benchmark was evaluated using a server with two Intel(R) Xeon(R) Platinum 8380 (IceLake) CPUs with 40 cores.

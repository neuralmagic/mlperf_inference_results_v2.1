# Neural Magic's BERT-Large DeepSparse MLPerf Submission

This is the repository of Neural Magic's BERT-Large DeepSparse submission for [MLPerf Inference Benchmark v2.1](https://www.mlperf.org/inference-overview/).

## Benchmark

In this submission, we show three different methods of optimizing BERT-Large by combining various methods of compression covering: unstructured gradual pruning, quantization-aware training, and structural distillation. 

Our MLPerf Inference v2.1 submission contains the following results for the BERT-Large SQuAD v1.1 question answering task:

| Benchmark            | % of BERT-Large FP32 Accuracy (F1=90.874) | Scenarios             |
|:--------------------:|:-----------------------------------------:|:---------------------:|
| [BERT-Large Prune OFA](prune-ofa_large.md) | 99.48% (F1=90.41)                         | SingleStream, Offline |
| [oBERT-Large](obert_large.md)          | 99.27% (F1=90.21)                         | SingleStream, Offline |
| [oBERT-MobileBERT](obert_mobilebert.md)     | 99.39% (F1=90.32)                         | SingleStream, Offline |

The benchmark implementation and models are stored in the [code/bert/deepsparse](code/bert/deepsparse) directory which contains a `README.md` detailing instructions on how to set up the benchmark. An example of the commands used to generate this submission is stored in [submission.sh](submission.sh).

The benchmark was evaluated using a server with two Intel(R) Xeon(R) Platinum 8380 (IceLake) CPUs with 40 cores each.

## Model Methods

[BERT-Large Prune OFA - Prune Once for All: Sparse Pre-Trained Language Models](prune-ofa_large.md)

[oBERT-Large: The Optimal BERT Surgeon applied to the BERT-Large model](obert_large.md)

[oBERT-MobileBERT: The Optimal BERT Surgeon applied to the MobileBERT model](obert_mobilebert.md)

## Citation info
If you find our models useful, please consider citing our work:
```bibtex
@article{zafrir2021prune,
  title={Prune Once for All: Sparse Pre-Trained Language Models},
  author={Zafrir, Ofir and Larey, Ariel and Boudoukh, Guy and Shen, Haihao and Wasserblat, Moshe},
  journal={arXiv preprint arXiv:2111.05754},
  year={2021}
}
```

```bibtex
@article{kurtic2022optimal,
  title={The Optimal BERT Surgeon: Scalable and Accurate Second-Order Pruning for Large Language Models},
  author={Kurtic, Eldar and Campos, Daniel and Nguyen, Tuan and Frantar, Elias and Kurtz, Mark and Fineran, Benjamin and Goin, Michael and Alistarh, Dan},
  journal={arXiv preprint arXiv:2203.07259},
  year={2022}
}
```

# obert-large: The Optimal BERT Surgeon applied to the BERT-Large model

[The Optimal BERT Surgeon: Scalable and Accurate Second-Order Pruning for Large Language Models](https://arxiv.org/abs/2203.07259) (oBERT) stands for an efficient and accurate weight pruning method based on approximate second-order information, which we showed to yield state-of-the-art results in both stages of language tasks: pre-training and fine-tuning. For the MLPerf inference submission, we apply it in the fine-tuning stage on the SQuADv1.1 task. More specifically we adopt the gradual downstream pruning setup presented in the paper and progressively prune and fine-tune the popular [bert-large-uncased](https://huggingface.co/bert-large-uncased) model with minor updates to hyper-parameters configuration.

High-performance inference usually benefits more from (semi) structured sparsity patterns than from the unstructured ones. Hence, we employ the generalized oBERT formulation introduced in the paper and prune weights in the 4-block pattern, meaning that contiguous blocks of 4 weights are either set to zero or kept dense. Both pruning types, unstructured and 4-block, can be leveraged for computational speedups with the DeepSparse runtime, but 4-block pruning coupled with INT8 quantization can provide further performance gains. For quantization, we apply standard quantization-aware training QAT on top of the 4-block pruned models.

To ease reproducibility, we conduct our experiments in popular open-source libraries: [Transformers](https://github.com/huggingface/transformers) and [SparseML](https://github.com/neuralmagic/sparseml). As previously noted, our compression setup consists of two steps, pruning and quantization, and now we present in detail each one of them with the corresponding configuration files (pruning *recipes*) and bash scripts to reproduce our results. For a fair comparison with other approaches we don't employ any form of structured compression and keep the original BERT-Large architecture intact.

## 1st step: semi-structured gradual pruning
Following the gradual pruning setup from the paper, we progressively prune the BERT-Large model over the span of 30-epochs. More specifically, we make use of the knowledge-distillation from the dense teacher, learning rate scheduler with rewinds and cubic sparsity scheduler with high initial pruning step. We prune the encoder part of the model in the semi-structured 4-block pattern up to the 95% sparsity.

Assuming that the SparseML library is installed, the bash script to reproduce our pruning setup is as follows:
```shell
CUDA_VISIBLE_DEVICES=0 src/sparseml/transformers/question_answering.py
    --distill_teacher neuralmagic/bert-large-uncased-finetuned-squadv1 \
    --model_name_or_path neuralmagic/bert-large-uncased-finetuned-squadv1 \
    --dataset_name squad \
    --do_train \
    --fp16 \
    --do_eval \
    --optim adamw_torch \
    --evaluation_strategy epoch \
    --save_strategy epoch \
    --save_total_limit 1 \
    --per_device_train_batch_size 16 \
    --per_device_eval_batch_size 16 \
    --learning_rate 1e-4 \
    --max_seq_length 384 \
    --doc_stride 128 \
    --preprocessing_num_workers 8 \
    --seed 42 \
    --num_train_epochs 30 \
    --recipe obert_large_pruning_recipe.yaml \
    --output_dir my_pruning_output_dir
```

And the *obert_large_pruning_recipe.yaml* is as follows:
```yaml
modifiers:
  - !EpochRangeModifier
    start_epoch: 0.0
    end_epoch: 30.0

training_modifiers:
  - !TrainableParamsModifier
    params:
    - bert.embeddings.word_embeddings.weight
    - bert.embeddings.position_embeddings.weight
    - bert.embeddings.token_type_embeddings.weight
    trainable: False
    params_strict: True
    start_epoch: 0.0
    end_epoch: 30.0

  - !LearningRateFunctionModifier
    start_epoch: 0.0
    end_epoch: 2.0
    lr_func: linear
    init_lr: 1e-4
    final_lr: 1e-6

  - !LearningRateFunctionModifier
    start_epoch: 2.0
    end_epoch: 28.0
    lr_func: cyclic_linear
    cycle_epochs: 2.0
    init_lr: 1e-4
    final_lr: 5e-5

  - !LearningRateFunctionModifier
    start_epoch: 28.0
    end_epoch: 30.0
    lr_func: linear
    init_lr: 1e-4
    final_lr: 1e-6

  - !OBSPruningModifier
    params: [
      "re:bert.encoder.layer.*.attention.self.query.weight",
      "re:bert.encoder.layer.*.attention.self.key.weight",
      "re:bert.encoder.layer.*.attention.self.value.weight",
      "re:bert.encoder.layer.*.attention.output.dense.weight",
      "re:bert.encoder.layer.*.intermediate.dense.weight",
      "re:bert.encoder.layer.*.output.dense.weight",
    ]
    init_sparsity: 0.7
    final_sparsity: 0.95
    start_epoch: 2.0
    end_epoch: 28.0
    update_frequency: 2.0
    inter_func: cubic
    global_sparsity: True
    mask_type: block4
    num_grads: 1024
    damp: 1e-8
    fisher_block_size: 4
    grad_sampler_kwargs:
      batch_size: 8

distillation_modifiers:
  - !DistillationModifier
     hardness: 1.0
     temperature: 5.0
     distill_output_keys: [start_logits, end_logits]
```

## 2nd step: quantization-aware training
Now that we have 95% semi-structured pruned BERT-Large model, we can apply INT8 quantization-aware training (QAT) on top of it to further improve the performance.
Assuming that the SparseML library is installed, the bash script to reproduce our quantization setup is as follows:

```shell
CUDA_VISIBLE_DEVICES=0 src/sparseml/transformers/question_answering.py
    --distill_teacher neuralmagic/bert-large-uncased-finetuned-squadv1 \
    --model_name_or_path /path/to/the/pruned/checkpoint/from/the/previous/step \
    --dataset_name squad \
    --do_train \
    --do_eval \
    --optim adamw_torch \
    --evaluation_strategy epoch \
    --save_strategy epoch \
    --save_total_limit 1 \
    --per_device_train_batch_size 16 \
    --per_device_eval_batch_size 16 \
    --learning_rate 2e-5 \
    --max_seq_length 384 \
    --doc_stride 128 \
    --preprocessing_num_workers 8 \
    --seed 42 \
    --num_train_epochs 2 \
    --recipe obert_large_quantization_recipe.yaml \
    --output_dir my_quantization_output_dir
```

And the *obert_large_quantization_recipe.yaml* is as follows:
```yaml
modifiers:
  - !EpochRangeModifier
    start_epoch: 0.0
    end_epoch: 2.0

  - !LearningRateFunctionModifier
    start_epoch: 0.0
    end_epoch: 2.0
    lr_func: linear
    init_lr: 2e-5
    final_lr: 0.0

pruning_modifiers:
  - !ConstantPruningModifier
    start_epoch: 0.0
    end_epoch: 2.0
    params: [
      "re:bert.encoder.layer.*.attention.self.query.weight",
      "re:bert.encoder.layer.*.attention.self.key.weight",
      "re:bert.encoder.layer.*.attention.self.value.weight",
      "re:bert.encoder.layer.*.attention.output.dense.weight",
      "re:bert.encoder.layer.*.intermediate.dense.weight",
      "re:bert.encoder.layer.*.output.dense.weight",
    ]

distillation_modifiers:
  - !DistillationModifier
     hardness: 1.0
     temperature: 5.0
     distill_output_keys: [start_logits, end_logits]

quantization_modifiers:
  - !QuantizationModifier
    activation_bits: 8
    disable_quantization_observer_epoch: 1.0
    exclude_batchnorm: True
    exclude_module_types: ['LayerNorm', 'BatchNorm1d', 'BatchNorm2d', 'BatchNorm3d']
    freeze_bn_stats_epoch: 1.0
    model_fuse_fn_name: conv_bn_relus
    quantize_conv_activations: True
    quantize_embeddings: True
    quantize_embedding_activations: True
    quantize_linear_activations: False
    reduce_range: False
    start_epoch: 0.0
    submodules: ['bert.encoder', 'qa_outputs', 'bert.embeddings']
    tensorrt: False
    weight_bits: 8
```

## Final step: export to ONNX and run with DeepSparse
To run the pruned and quantized `obert-large` model in the DeepSparse engine, we need to export it to ONNX first:
```shell
sparseml.transformers.export_onnx \
    --model_path /path/to/my/pruned/and/quantized/model \
    --task 'question-answering' --sequence_length 384
```

TODO: Michael please add the command to run this model with DeepSparse

## Additional info

For more details about our compression approach, please check the Optimal BERT Surgeon (oBERT) paper: [https://arxiv.org/abs/2203.07259](https://arxiv.org/abs/2203.07259).

The Optimal BERT Surgeon code with more recipes, examples and tutorials: [https://github.com/neuralmagic/sparseml/tree/main/research/optimal_BERT_surgeon_oBERT](https://github.com/neuralmagic/sparseml/tree/main/research/optimal_BERT_surgeon_oBERT).

## Citation info
If you find the model useful, please consider citing our work:
```bibtex
@article{kurtic2022optimal,
  title={The Optimal BERT Surgeon: Scalable and Accurate Second-Order Pruning for Large Language Models},
  author={Kurtic, Eldar and Campos, Daniel and Nguyen, Tuan and Frantar, Elias and Kurtz, Mark and Fineran, Benjamin and Goin, Michael and Alistarh, Dan},
  journal={arXiv preprint arXiv:2203.07259},
  year={2022}
}
```


# 代码

[How to train a custom seq2seq model with BertModel](https://github.com/huggingface/transformers/issues/4517)

```python
import os
os.environ['USE_TORCH']="1"
import torch
from transformers import EncoderDecoderModel,AutoTokenizer

model = EncoderDecoderModel.from_encoder_decoder_pretrained('bert-base-chinese', 'bert-base-chinese') # initialize Bert2Bert
tokenizer = AutoTokenizer.from_pretrained('bert-base-chinese')
```

```python
tokenizer.save_pretrained('/home/lcz/model_data/my_seq2seq/')

```


/home/lcz/model_data/my_seq2seq


python examples/pytorch/summarization/run_summarization.py \
    --model_name_or_path ../model_data/my_seq2seq/ \
    --do_train \
    --do_eval \
    --dataset_name cnn_dailymail \
    --dataset_config "3.0.0" \
    --source_prefix "summarize: " \
    --output_dir /tmp/tst-summarization \
    --per_device_train_batch_size=4 \
    --per_device_eval_batch_size=4 \
    --overwrite_output_dir \
    --predict_with_generate


python examples/pytorch/summarization/run_summarization.py \
--model_name_or_path ../model_data/my_seq2seq/ \
--do_train \
--do_eval \
--train_file ../seq2seq_tra.json \
--validation_file ../seq2seq_tra.json \
--text_column text \
--summary_column summary \
--output_dir ../model_data/tmp-summarization \
--per_device_train_batch_size=4 \
--per_device_eval_batch_size=4 \
--overwrite_output_dir \
--predict_with_generate




#

pip install rouge_score



# 参考：

- [How to train a custom seq2seq model with BertModel](https://github.com/huggingface/transformers/issues/4517)
- [Hugging Face model card:bert2bert-cnn_dailymail-fp16](https://huggingface.co/patrickvonplaten/bert2bert-cnn_dailymail-fp16)
- [colab tutorial: how to warm-start a BERT2BERT model](https://colab.research.google.com/drive/1WIk2bxglElfZewOHboPFNj8H44_VAyKE?usp=sharing)
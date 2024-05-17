# ONNX 리서치

해당 문서는 ML 모델 최적화를 위한 Tensorflows/Torch 모델의 ONNX + Optimum 사용법에 대해 다룹니다.

# 1. ONNX 변환

크게 세 가지 방법으로 변환할 수 있습니다.

## 1.1. Torch.onnx.export

torch.onnx.export 함수를 사용하여 변환하는 방법입니다. Transformers가 아닌 torch 모델에도 적용할 수 있지만, dummy input value를 줘야 하고 설정 자체를 복잡하게 해야 되기 때문에 추천드리지 않습니다.

```python
import torch
from transformers import AutoModelForSequenceClassification, AutoTokenizer

# load model and tokenizer
model_id = "distilbert-base-uncased-finetuned-sst-2-english"
model = AutoModelForSequenceClassification.from_pretrained(model_id)
tokenizer = AutoTokenizer.from_pretrained(model_id)
dummy_model_input = tokenizer("This is a sample", return_tensors="pt")

# export
torch.onnx.export(
    model, 
    tuple(dummy_model_input.values()),
    f="torch-model.onnx",  
    input_names=['input_ids', 'attention_mask'], 
    output_names=['logits'], 
    dynamic_axes={'input_ids': {0: 'batch_size', 1: 'sequence'}, 
                  'attention_mask': {0: 'batch_size', 1: 'sequence'}, 
                  'logits': {0: 'batch_size', 1: 'sequence'}}, 
    do_constant_folding=True, 
    opset_version=13, 
)
```

## 1.2. transformers.onnx

huggingface 자체 onnx 기능을 사용하는 방법입니다. 이 때, transformer는 추가 디펜던시가 필요합니다.

```python
pip install transformers[onnx]
```

설치 이후에는 기존 모델을 좀 더 간단히 변환할 수 있습니다.

```python
from pathlib import Path
import transformers
from transformers.onnx import FeaturesManager
from transformers import AutoConfig, AutoTokenizer, AutoModelForSequenceClassification

# load model and tokenizer
model_id = "distilbert-base-uncased-finetuned-sst-2-english"
feature = "sequence-classification"
model = AutoModelForSequenceClassification.from_pretrained(model_id)
tokenizer = AutoTokenizer.from_pretrained(model_id)

# load config
model_kind, model_onnx_config = FeaturesManager.check_supported_model_or_raise(model, feature=feature)
onnx_config = model_onnx_config(model.config)

# export
onnx_inputs, onnx_outputs = transformers.onnx.export(
        preprocessor=tokenizer,
        model=model,
        config=onnx_config,
        opset=13,
        output=Path("trfs-model.onnx")
)
```

## 1.3. Optimum

Optimum이라는 오픈소스 라이브러리를 사용하여 변환하는 방법입니다. 이 경우, 지원하는 모델들이 제한되어 있으나 아주 간단하게 변환 가능합니다. 이후 모든 과정은 해당 라이브러리를 사용하여 진행할 예정입니다.

```python
pip install optimum[onnxruntime]
```

```python
from optimum.onnxruntime import ORTModelForSequenceClassification

model = ORTModelForSequenceClassification.from_pretrained("distilbert-base-uncased-finetuned-sst-2-english",from_transformers=True)
model.save_pretrained("model.onnx")
```

# 2. Onnx Quantization

Onnx 모델들은 Optimum 라이브러리를 통해 손쉽게 quantization이 가능하며, 몇 가지 strategy 중에서 선택하여 변환 가능합니다. Onnx Quantization을 통해 CPU 실행 속도를 많이 줄일 수 있으며, GPU 리소스가 부족한 경우 대안으로 사용할 수 있습니다. 여기에서는 avx512_vnni로 진행합니다.

## 2.1. Encoder-only model

기본적으로 한 개의 그래프로 구성된 모델의 경우 간단히 quantize를 수행할 수 있습니다.

```python
from optimum.onnxruntime import ORTQuantizer, ORTModelForSequenceClassification
from optimum.onnxruntime.configuration import AutoQuantizationConfig

onnx_model = ORTModelForSequenceClassification.from_pretrained("distilbert-base-uncased-finetuned-sst-2-english", export=True)

quantizer = ORTQuantizer.from_pretrained(onnx_model)

dqconfig = AutoQuantizationConfig.avx512_vnni(is_static=False, per_channel=False)

model_quantized_path = quantizer.quantize(
    save_dir="path/to/output/model",
    quantization_config=dqconfig,
)
```

## 2.2. Seq2Seq model

seq2seq 모델의 경우 optimum을 사용했을 때 기본적으로 세 개의 onnx 파일이 나옵니다. [encoder, decoder, 그리고 decoder with past model] 해당 모델들은 한 번에 quantize하는 걸 지원하지 않기 때문에, 각각 quantize해 주어야 합니다.

```python
from optimum.onnxruntime import ORTModelForSeq2SeqLM, ORTQuantizer

model_folder = "/home/hayes/name_relevant/legacy"
tokenizer = AutoTokenizer.from_pretrained(model_folder)
model = ORTModelForSeq2SeqLM.from_pretrained(model_folder, from_transformers=True)
model_dir = "model.onnx"
encoder_quantizer = ORTQuantizer.from_pretrained(model_dir, file_name="encoder_model.onnx")
decoder_quantizer = ORTQuantizer.from_pretrained(model_dir, file_name="decoder_model.onnx")
decoder_wp_quantizer = ORTQuantizer.from_pretrained(model_dir, file_name="decoder_with_past_model.onnx")

quantizer = [encoder_quantizer, decoder_quantizer, decoder_wp_quantizer]

from optimum.onnxruntime.configuration import AutoQuantizationConfig
dqconfig = AutoQuantizationConfig.avx512_vnni(is_static=False, per_channel=False)

for q in quantizer:
    q.quantize(save_dir="./quntized_model",quantization_config=dqconfig)  # doctest: +IGNORE_RESULT
```

# 3. onnx model 실행

onnx 모델의 경우 transformers 라이브러리의 pipeline 함수를 이용해서 inference가 가능합니다. 해당 문서에서는 onnx engine을 직접 사용하지 않고, onnx pipeline으로 추상화하여 사용하는 방법을 다룹니다.

```python
from optimum.onnxruntime import ORTModelForSeq2SeqLM
from transformers import pipeline, AutoTokenizer

# CPU inference
model = ORTModelForSeq2SeqLM.from_pretrained("quntized_model")

# GPU inference
model = ORTModelForSeq2SeqLM.from_pretrained("quntized_model")

tokenizer = AutoTokenizer.from_pretrained("/home/hayes/name_relevant/legacy")

# CPU
onnx_classifier = pipeline("text2text-generation", model=model, tokenizer=tokenizer)

# GPU
onnx_classifier = pipeline("text2text-generation", model=model, tokenizer=tokenizer, device="cuda:0")

# 실행
onnx_classifier("summarize: 세제 묶음 20개 *20|세제")
# [{'generated_text': '연관'}]
```

# 4. ONNX quantize 결과

model size : 3.1GB → 1.3GB

모델 성능 : (Name_relevant, Acc) 98.4% → 98.5%

CPU inference 속도 : (128개 기준) 5초 → 2초
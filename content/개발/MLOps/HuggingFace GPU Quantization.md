# HuggingFace GPU Quantization

GPU Quantization은 머신러닝 모델의 크기를 줄이고 실행 속도를 빠르게 하는 기술로, 이를 통해 GPU의 메모리 사용을 줄일 수 있습니다. 이 기술은 모델의 정확도를 손상시키지 않으면서 모델의 크기를 줄이는 데 주로 사용됩니다. 이 문서에서는 Int8 혼합 정밀도 행렬 분해로 성능을 거의 떨어트리지 않으면서 속도와 메모리를 아끼는 방법에 대해 소개합니다.

reference : [https://huggingface.co/docs/transformers/ko/perf_infer_gpu_one](https://huggingface.co/docs/transformers/ko/perf_infer_gpu_one)

노트북 : [http://172.16.10.201:8889/notebooks/transformers_int8_quantization.ipynb](http://172.16.10.201:8889/notebooks/transformers_int8_quantization.ipynb) / fd56b464fb94d37cfd55975e2cb300c2a7e50b82d83d550b

<aside>
💡 커널은 GPU 전용으로 컴파일되어 있기 때문에 해당 최적화를 사용하여 실행할 경우 GPU가 필요합니다. GPU가 아닌 CPU에서의 최적화를 원하시는 경우, [ONNX 리서치](https://www.notion.so/ONNX-2342c57df3624dc1b573430530e2ed97?pvs=21) 문서를 참고해 주시기 바랍니다.

</aside>

## 0. 의존성

bitsandbytes와 accelerate 패키지가 필요합니다.

```bash
pip install bitsandbytes>=0.39.0 git+https://github.com/huggingface/accelerate.git
```

## 1. 모델 로드

의존성 설치 후, 모델을 로드할 때 `load_in_8bit=True` 옵션을 설정해주시면 됩니다. 해당 문서에서는 pko-T5-large 모델로 fine-tuning된 name-query-relevant 모델을 사용합니다.

```python
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM
tokenizer = AutoTokenizer.from_pretrained(model_folder)
model = AutoModelForSeq2SeqLM.from_pretrained(model_folder, device_map="auto", load_in_8bit=True)
```

## 2. 모델 인퍼런스

해당 quantization의 경우 인퍼런스 시 유의사항이 몇 가지 있습니다.

- `pipeline()` 함수 대신 모델의 `generate()` 메소드를 사용하는 것을 권장합니다. `pipeline()` 함수로도 추론은 가능하나 혼합 8비트 모델에 최적화되지 않아 더 느립니다. 또한 일부 샘플링 전략은 `pipeline()` 함수에서는 지원되지 않습니다.
- 입력을 모델과 동일한 GPU에 배치하는 것이 좋습니다.

```python
model_inputs = tokenizer(inps, max_length=src_max, padding=True, truncation=True, return_tensors='pt').to(torch.device('cuda'))
preds = model.generate(model_inputs['input_ids'],max_length=out_max)
out = tokenizer.batch_decode(preds, skip_special_tokens=True)
```

## 3. Quantization 결과

모델 로딩 RAM : 8000MB → **4000MB**

GPU VRAM : (batch_size=32) 4000MB → **3000MB**

inference 속도 : (batch_size=128) 0.47s → **0.35s**
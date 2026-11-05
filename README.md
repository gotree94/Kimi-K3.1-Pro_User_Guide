# Kimi K3.1 Pro (moonshotai/Kimi-K3.1-Pro) 다운로드 가이드

> 다른 PC에서 다운로드받기 위한 정리 문서
> Hugging Face: https://huggingface.co/moonshotai/Kimi-K3.1-Pro
> 작성일: 2026-11-05

---

## 1. 모델 개요

| 항목 | 내용 |
|---|---|
| 개발사 | Moonshot AI |
| 아키텍처 | Mixture-of-Experts (MoE), 네이티브 멀티모달 (텍스트 + 이미지) |
| 총 파라미터 | 2.8T (2,800,000,000,000) |
| 활성 파라미터 | 104B |
| 레이어 수 | 93 (Dense 1) |
| 어텐션 | 69 KDA + 24 Gated DSA |
| 전문가 수 | 896 (토큰당 16 선택, 공유 2) |
| 컨텍스트 길이 | 1,048,576 토큰 (약 100만) |
| 비전 인코더 | MoonViT-V2 (401M) |
| 양자화 | MXFP4 가중치 / MXFP8 활성 (QAT) |
| 라이선스 | Kimi K3.1 Pro License (상업적 사용 전 라이선스 확인 필요) |

---

## 2. 다운로드 용량 및 구성

- **총 다운로드 용량: 약 1.5 TB (≈ 1,545 GB)**
- Safetensors 샤드 **96개** (`model-00001-of-000096.safetensors` ~ `model-00096-of-000096.safetensors`)
- 샤드 1개당 약 16~17 GB
- 기타 파일: `config.json`, `tokenizer*`, `tiktoken.model`, `configuration_kimi_k3.py`, `kimi_k3_processor.py`, `README.md`, `LICENSE` 등

> ⚠️ **주의**
> - 단일 디스크에 1.5TB 이상 여유 공간이 필요합니다.
> - 다운로드 중 임시/캐시 공간까지 고려하면 **2TB 이상 여유**를 권장합니다.
> - 일반 PC(단일 GPU, 24GB VRAM 등)에서는 **실행 불가**합니다. 실행은 8×H100/B200급 서버가 필요합니다. (다운로드/보관만 하는 경우에는 무관)
> - 다운로드만 하고 실행하지 않을 경우, 이 문서의 **4~6번 항목만** 수행하면 됩니다.

---

## 3. 사전 준비

### 3.1 Python 및 huggingface_hub 설치

```bash
# Python 3.10 이상 권장
python --version

pip install -U "huggingface_hub[cli]" hf_transfer
```

- `hf_transfer`는 대용량 다운로드 속도를 크게 높여줍니다(권장).

### 3.2 (선택) Hugging Face 로그인

공개 모델이지만, 속도 제한 회피 및 향후 게이트 모델 대비 로그인 권장:

```bash
hf auth login
# 또는 구버전: huggingface-cli login
```

토큰 발급: https://huggingface.co/settings/tokens

### 3.3 다운로드 가속 활성화

Windows (PowerShell):

```powershell
$env:HF_HUB_ENABLE_HF_TRANSFER = "1"
```

Linux / macOS (bash):

```bash
export HF_HUB_ENABLE_HF_TRANSFER=1
```

---

## 4. 다운로드 명령

### 4.1 Windows (PowerShell)

```powershell
pip install -U "huggingface_hub[cli]" hf_transfer
$env:HF_HUB_ENABLE_HF_TRANSFER = "1"

hf download moonshotai/Kimi-K3.1-Pro --local-dir "D:\models\Kimi-K3.1-Pro"
```

### 4.2 Linux (bash)

```bash
pip install -U "huggingface_hub[cli]" hf_transfer
export HF_HUB_ENABLE_HF_TRANSFER=1

hf download moonshotai/Kimi-K3.1-Pro --local-dir /data/models/Kimi-K3.1-Pro
```

> 구버전 CLI를 쓰는 경우 `hf download` 대신 `huggingface-cli download`를 사용하세요.

### 4.3 전체 용량이 부족할 때 (샤드 일부만 받기)

특정 파일만 받으려면 파일명을 지정합니다. 예) config/토크나이저 + 1번 샤드만:

```bash
hf download moonshotai/Kimi-K3.1-Pro \
  --include "*.json" "*.py" "tiktoken.model" "model-00001-of-000096.safetensors" \
  --local-dir /data/models/Kimi-K3.1-Pro
```

---

## 5. 중단 후 재개 (Resume)

- `hf download`는 기본적으로 **이어받기(resume)** 를 지원합니다.
- 같은 `--local-dir`로 **동일한 명령을 다시 실행**하면 이미 받은 파일/부분은 건너뜁니다.
- 불안정한 네트워크에서는 백그라운드 + 재시도 루프 권장:

Linux:

```bash
until hf download moonshotai/Kimi-K3.1-Pro --local-dir /data/models/Kimi-K3.1-Pro; do
  echo "재시도..."; sleep 10
done
```

Windows (PowerShell):

```powershell
do {
  hf download moonshotai/Kimi-K3.1-Pro --local-dir "D:\models\Kimi-K3.1-Pro"
} while ($LASTEXITCODE -ne 0)
```

---

## 6. 다운로드 검증

### 6.1 파일 개수/용량 확인

Windows (PowerShell):

```powershell
$dir = "D:\models\Kimi-K3.1-Pro"
(Get-ChildItem -LiteralPath $dir -Filter "*.safetensors").Count   # 96 이어야 함
"{0:N1} GB" -f ((Get-ChildItem -LiteralPath $dir -Recurse | Measure-Object Length -Sum).Sum / 1GB)
```

Linux:

```bash
ls /data/models/Kimi-K3.1-Pro/*.safetensors | wc -l    # 96
du -sh /data/models/Kimi-K3.1-Pro
```

### 6.2 무결성 검증 (선택, 시간 소요)

```bash
hf download moonshotai/Kimi-K3.1-Pro --local-dir /data/models/Kimi-K3.1-Pro --verify
```

> 검증은 수 TB 읽기/해시 연산이 필요하므로 시간이 오래 걸립니다. 정상 완료 메시지가 떴다면 생략 가능.

---

## 7. (참고) 실행 — 서버급 장비에서만

이 모델은 로컬 PC에서 실행 불가하며, 아래 엔진 + 멀티 GPU 서버가 필요합니다.

- vLLM: https://github.com/vllm-project/vllm (recipes: https://recipes.vllm.ai/moonshotai/Kimi-K3.1-Pro)
- SGLang: https://github.com/sgl-project/sglang (cookbook: https://docs.sglang.io/cookbook/autoregressive/Moonshotai/Kimi-K3.1-Pro)
- TokenSpeed: https://github.com/lightseekorg/tokenspeed

vLLM 예시:

```bash
pip install vllm
vllm serve "moonshotai/Kimi-K3.1-Pro" \
  --tensor-parallel-size 8 \
  --trust-remote-code
```

- `reasoning_effort`: `"low"` | `"high"` | `"max"` (기본 `max`)
- 멀티턴/툴 호출 시 어시스턴트 메시지의 `reasoning_content`, `tool_calls`를 **그대로** 다시 `messages`에 넘겨야 합니다.

---

## 8. 다운로드 없이 쓰는 대안: API

로컬 다운로드/실행이 부담되면 공식 API 사용:

- Kimi Platform: https://platform.kimi.ai
- 모델 ID: `kimi-k3.1-pro` (OpenAI/Anthropic 호환 API 제공)
- Quickstart: https://platform.kimi.ai/docs/guide/kimi-k3-1-pro-quickstart

---

## 9. 체크리스트

- [ ] 대상 PC 디스크 여유 공간 2TB 이상 확보
- [ ] Python 3.10+ 및 `huggingface_hub[cli]`, `hf_transfer` 설치
- [ ] (권장) `hf auth login` 완료
- [ ] `HF_HUB_ENABLE_HF_TRANSFER=1` 설정
- [ ] `hf download ... --local-dir <경로>` 실행
- [ ] safetensors 샤드 96개 / 약 1.5TB 확인
- [ ] (선택) `--verify`로 무결성 검증

---

## 10. 참고 링크

- 모델 페이지: https://huggingface.co/moonshotai/Kimi-K3.1-Pro
- 파일 목록: https://huggingface.co/moonshotai/Kimi-K3.1-Pro/tree/main
- 기술 블로그: https://www.kimi.ai/blog/kimi-k3-1-pro
- 라이선스: https://huggingface.co/moonshotai/Kimi-K3.1-Pro/blob/main/LICENSE

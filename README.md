# AI Image Generator for Industry Field
### 현업 디자이너를 위한 이미지 생성 파이프라인

연세대학교 산업공학과 산업인공지능특론 프로젝트  
**Korean Prompt → Image Generation → Background Removal (End-to-End)**


## Overview

본 프로젝트는 디자인 전담 조직의 리소스가 제한적인 산업군을 대상으로(금융 등), 현업 실무자가 자연어 질의(한국어)만으로도 즉시 활용 가능한 디자인 이미지를 생성할 수 있도록 하는  
**End-to-End 이미지 생성 파이프라인**을 제안

기존 Text-to-Image 모델은 영어 프롬프트 중심, GPU 의존적 구조로 폐쇄망 및 내부망 환경에서 활용이 어렵고, 확률 기반 생성으로 인한 스타일 유지 어려움 등의 문제가 존재

==> 본 프로젝트에서는 **번역–생성–후처리**를 하나의 파이프라인으로 통합
일부 모듈을 **CPU 기반**으로 설계하여 실무 적용 가능성을 높임



## Motivation

### 산업계 디자인 작업의 한계
- 금융, 반도체, 제약, 물류 등 디자인 우선순위가 낮은 산업군
- 디자인 전담 조직은 존재하나 인력·시간 제약
- 반복적이고 단순한 요청(기존 오브제 변형)이 다수

### 문제 인식
- “새로운 창작”보다 기존 디자인과 자연스럽게 이어지는 이미지가 필요
- 영어 프롬프트 기반 모델은 현업 접근성이 낮음
- 내부망 환경에서 실행 가능한 경량 파이프라인 필요


## Proposed Pipeline
'''
Korean Prompt
↓
Korean → English Translation (BART)
↓
Text-to-Image Generation (Stable Diffusion)
↓
Salient Object Detection (U²-Net)
↓
Background Removed Image
'''
![Demo Image](demo/image.png)

**입력 예시**
“자동차를 타고 있는 캐릭터 이미지 생성해줘”

### Input
- 한국어 자연어 프롬프트

### Output
- 배경이 제거된 최종 이미지 (실사용 가능)

### Components

#### Translation Module
- BART 기반 한–영 번역 모델

#### Text-to-Image Module
- Stable Diffusion v2.1 (DreamBooth fine-tuning)

#### Post-processing Module
- U²-Net 기반 Salient Object Detection (배경제거)

---

## Translation Model (KOR → ENG)

- **Model**: BART-base
- **Data**
  - 기술과학 / 사회과학 / 일상 / 전문(식품) 번역 데이터
  - 총 820만 문장
- **Evaluation**
  - ROUGE-1 / ROUGE-2 / ROUGE-L
- **Optimization**
  - ONNX 변환
  - INT8 Quantization
  - CPU 추론 지원


## Text-to-Image Generation

### Base Model
- Stable Diffusion v2.1
  - CLIP Text Encoder
  - VAE
  - UNet


### Fine-tuning
- DreamBooth 방식
 일부 레이어만 fine-tuning하여
- 사전 학습 지식 유지
- 과적합 방지

### 학습 데이터
- 캐릭터 (30장), 자동차(10장), 카트(4장), 섬(12장), 주유소(3장), 코인(6장) 등 
- Midjourney로 생성
- **신한 카드 캐릭터 및 신한 카드 어플 내 icon 사용**

## Background Removal

- **Model**: U²-Net
- **Task**: Salient Object Detection

### 특징
- 가장 주목되는 객체만 분리
- 복잡한 배경 제거에 효과적

### 실행 환경
- ONNX Runtime
- CPU 기반 추론

---

## Results

### Scheduler 비교
- DDIM, DPM++, Euler, Euler Ancestral, LMS 등 9종 비교
- **평가 방식**: MOS (Mean Opinion Score)

### 주요 결과
- **Euler Ancestral**
- 이미지 품질·유사성 우수
- **LMS**
- 텍스트 프롬프트 반영 능력 우수

→ 디자인 스타일 유지 + 객체 변형 작업에 적합

---

## Real-world Inference Examples

- 의상 변경 (Dress-up)
- 차량/카트 탑승 캐릭터
- 소품(동전 등) 상호작용 이미지

→ 기존 디자인 스타일을 유지하면서 새로운 요소를 자연스럽게 결합

---

## System Characteristics

- End-to-End 자동화 파이프라인
- 한국어 질의 지원
- CPU 서빙 가능 (번역 / 배경 제거)
- 폐쇄망·내부망 환경 적용 가능
- 반복 디자인 작업 시간 단축


## Demo

- 데모 시연은 강의 발표 자료 및 노트북 기반 실행
- 한국어 입력 → 이미지 생성 → 배경 제거까지 확인 가능
# 🤖 AI 강사 에이전트 구축 프로젝트

본 프로젝트는 **LangGraph**를 활용하여, PPT 파일만 업로드하면 강의 스크립트 작성, RAG를 통한 정보 보강, 영상/음성 합성을 거쳐 **강의 영상과 복습 퀴즈까지 자동으로 생성**하는 AI 강사 에이전트를 개발한 사례입니다.

저는 이 프로젝트에서 학습자의 능동적인 학습 경험을 위한 **①복습 퀴즈 생성(`Make_quiz`)** 로직 개발과, LLM이 강의 흐름을 더 잘 이해하도록 돕는 **②강의 스크립트 생성 고도화** 작업에 참여했습니다.

---

## 📌 프로젝트 개요

* **목표**: LangGraph와 LLM을 활용하여, PPT 파일 분석, RAG 기반 스크립트 생성, TTS/영상 합성을 거쳐 최종적으로 **강의 내용 기반의 복습 퀴즈**까지 자동으로 생성하는 End-to-End AI 강사 에이전트 개발
* **진행 기간**: 2025.11.10 ~ 2025.11.13 (4일)
* **참여 인원**: 7
  
---

## 👥 팀 구성 및 개인 역할

| 역할 | 담당자 | 주요 업무 |
| :--- | :--- | :--- |
| **퀴즈 생성 / 스크립트 고도화** | **본인** | - **복습 퀴즈 생성 로직 개발:** `Make_quiz` (`node_generate_quiz`) 노드 개발. 전체 강의 스크립트(`total_script`) 기반 퀴즈/정답 생성<br>- **강의 스크립트 생성 고도화 (일부):** `Gen_script_ctx` 노드의 프롬프트를 수정하여, 이전 슬라이드 문맥을 참조하고 슬라이드 간의 연결성을 강화하는 작업 참여 |
| **PM / Gradio** | 팀원 A | - 과제 총괄 진행, State, Graph 담당<br>- **Gradio 웹 인터페이스 제작** 및 에이전트 연동 |
| **정보분해 고도화** | 팀원 B | - `Parse_all` 노드: PPT 내 텍스트, 이미지, 표, **도형** 정보 추출 고도화 |
| **페이지별 내용 생성** | 팀원 C | - `Tool_search`, `Gen_page` 노드: RAG 기반 외부 정보 검색 및 슬라이드별 내용 생성 |
| **강의 스크립트 생성** | 팀원 D | - `Gen_script_ctx` 노드: 슬라이드 내용 기반 강의 스크립트 생성 총괄 |
| **반복/종료/합성** | 팀원 E | - `Acc_step` (조건부 분기), `concat` (최종 영상 합치기) 노드 개발 |
| **영상 제작** | 팀원 F | - `tts`, `Make_video` 노드: TTS 변환 및 MoviePy 기반 슬라이드별 영상 합성 |

> 🔹 본 프로젝트는 팀원 전체의 긴밀한 협업으로 진행되었습니다. 특히 LangGraph의 복잡한 상태(State)를 일관되게 관리하기 위해, Notion으로 실시간으로 공유하고 검증하며 개발을 진행했습니다.

---

## 🛠 기술 스택 (Tech Stack)

| 분야 | 도구 / 라이브러리 |
| :--- | :--- |
| 언어 | Python |
| LMOps / Agent | **LangChain**, **LangGraph**, OpenAI (GPT-4o-mini) |
| 데이터 / 파싱 | RAG (Chroma DB), python-pptx |
| UI / 환경 | **Gradio**, Google Colab |
| 영상 / 음성 | gTTS, MoviePy, FFmpeg |

---

## 👣 프로젝트 주요 프로세스



| 단계 | 내용 |
| :--- | :--- |
| **v1.0 분석** | `Step1` (개인 과제): 단일 슬라이드 기반의 정적 강의 영상 생성기 분석 |
| **v2.0 설계** | **LangGraph** 기반 다중 슬라이드, RAG, **퀴즈 기능**이 포함된 순환(Loop) 에이전트 구조 설계 |
| **핵심 로직 구현** | `Parse_all` (정보 분해) → `Tool_search` (RAG) → `Gen_page` (내용 생성) → `Gen_script_ctx` (스크립트 생성) → `tts`/`Make_video` (영상/음성 합성)의 순환 구조 개발 |
| **로직 고도화 (본인/팀)** | `Gen_script_ctx` (스크립트) 프롬프트 고도화 참여, `Make_quiz` (퀴즈) 노드 신규 개발 |
| **UI 연동** | **Gradio**를 활용하여 파일 업로드, 옵션 선택, 영상 시청, 복습 퀴즈 풀이가 가능한 웹 인터페이스 구현 |

---

## ✨ 주요 특징 및 구현 내용 

### ✅ 1. 'Make_quiz' 노드 개발: LLM 기반 복습 퀴즈 생성

학습자가 강의 내용을 효과적으로 점검하고 능동적인 학습 경험을 하도록, LangGraph의 `Make_quiz` 노드 개발을 담당했습니다.

* **전체 스크립트 기반 생성**: 에이전트가 모든 슬라이드의 강의 스크립트 생성을 완료하면, State에 누적된 `total_script` 전체를 입력으로 받습니다.
* **LLM 프롬프트 엔지니어링**: LLM이 스크립트의 핵심 내용을 바탕으로 **자가진단용 객관식 퀴즈**를 생성하도록 프롬프트를 설계했습니다. 강의의 핵심 키워드를 놓치지 않고 문제에 반영하는 데 집중했습니다.
* **안정적인 상태(State) 관리**: 퀴즈 생성은 모든 슬라이드 처리가 끝난 *이후* 단 한 번만 실행되어야 했습니다. `Acc_step` 노드(조건부 엣지)가 `DONE`을 반환할 때만 `Make_quiz` 노드가 트리거되도록 그래프 흐름을 설계하고 디버깅했습니다.

### ✅ 2. 강의 스크립트 생성 고도화 참여 (`Gen_script_ctx`)

강의 스크립트의 품질을 높이고 슬라이드 간의 흐름을 자연스럽게 연결하기 위해 `Gen_script_ctx` 노드의 프롬프트 고도화 작업에 참여했습니다.

* **문맥(Context) 주입**: 기존 노드가 현재 슬라이드 정보에만 집중했던 것을 개선하여, State에 저장된 **이전 슬라이드의 핵심 요약(`previous_script_summary`)**을 함께 입력받도록 프롬프트를 수정했습니다.
* **연결성(Coherence) 강화**: "앞서 말씀드린...", "자, 그럼 다음으로..."와 같이, 이전 내용을 자연스럽게 참조하고 다음 내용으로 넘어가는 **연결 표현을 생성**하도록 LLM을 유도하여 전체 강의 스크립트의 흐름이 매끄럽게 이어지도록 개선했습니다.

### ✅ 3. (Project) LangGraph 기반 순환형 에이전트

팀 프로젝트의 핵심 로직인 LangGraph의 순환(Loop) 구조를 이해하고, 저의 모듈이 이 흐름에 올바르게 연동되도록 했습니다.

* **흐름**: (1)`Parse_all` (전체 슬라이드 분해) → (2)`Tool_search`/`Gen_page`/`Gen_script_ctx`... (슬라이드별 순환) → (3)`Acc_step` (반복 또는 종료 결정)
* **연동**: 저의 `Gen_script_ctx` 고도화가 `(2)번` 내부 루프에서, `Make_quiz` 노드가 `(3)번`의 루프 종료(`DONE`) 시점에 정확히 실행되도록 팀원들과 State를 관리하고 그래프 엣지를 구성했습니다.

---

## 📈 주요 개선 사항 (v1.0 → v2.0)

| 구분 | v1.0 (기존 한계) | v2.0 (주요 개선 사항) |
| :--- | :--- | :--- |
| **처리 범위** | 단일 슬라이드 기반 | **다중 슬라이드** 동시 처리 및 슬라이드 간 연결성 강화 |
| **콘텐츠 품질** | 슬라이드 내부 텍스트만 반영 | **RAG(`Tool_search`)** 도입으로 외부 지식(표, 이미지 정보 포함) 보강 |
| **학습 경험** | 강의 영상의 일방적 시청 | **복습 퀴즈(`Make_quiz`) 기능 추가**로 능동적인 학습 경험 유도 |
| **사용자 맞춤** | 기능 없음 | 말투, 목소리, 재생 속도 등 **사용자 맞춤 옵션** 제공 |
| **사용성** | Jupyter Notebook 환경에서만 실행 | **Gradio 웹 인터페이스**를 구현하여 누구나 쉽게 사용 |

---

## Gradio 결과

[Gradio 인터페이스]

![Gradio 인터페이스](https://github.com/Kim-geun-woo/AI-Instructor--Agent--Development/blob/main/images/Gradio%20Interface.png)

[Gradio에서 생성된 영상, 퀴즈]

![Gradio 영상, 퀴즈](https://github.com/Kim-geun-woo/AI-Instructor--Agent--Development/blob/main/images/Gradio%20quiz%2C%20lecture.png)

---

## 🎥 프로젝트 시연 영상 (MP4)

[![AI 강사 시연 영상](https://github.com/Kim-geun-woo/AI-Instructor--Agent--Development/raw/main/images/final.png)](https://youtu.be/W7wCsGLEiKM)
> 이미지를 클릭하면 유튜브로 이동합니다.

---

## 💡 회고 (What I Learned)

* **State 관리의 중요성**: LangGraph 프로젝트의 성패는 **State**를 얼마나 잘 설계하고 관리하느냐에 달려있음을 깨달았습니다. 팀원 전체가 Notion으로 State의 입출력을 공유했던 경험은 디버깅 시간을 획기적으로 줄이는 핵심이었습니다. 이는 복잡한 에이전트의 흐름을 제어하는 가장 중요한 역량임을 배웠습니다.

* **협업과 모듈화**: 저는 `Make_quiz`와 `Gen_script_ctx`의 *로직*을, PM(팀원 A)은 *Gradio UI*를 담당하는 등 명확한 역할 분담을 경험했습니다. 제가 개발한 퀴즈 생성 모듈(함수)이 LangGraph 노드로 통합되고, 이것이 다시 다른 팀원이 만든 UI와 `State`를 통해 연동되는 과정을 통해 모듈화된 개발과 인터페이스 정의의 중요성을 배웠습니다.

* **최종 사용자 고려**: 퀴즈 기능은 단순한 '기능 추가'가 아니라, 사용자의 '학습 경험'을 위한 장치로 에이전트 개발 뿐만 아니라 모든 개발에서 최종 산출물은 최종 사용자의 경험을 고려하는 것의 중요성을 배웠습니다.

---

## 📂 프로젝트 산출물

* `AI 미프 3차_05반_09조.pptx` (최종 발표 자료)
* `Step1_개인과제_AI_강사_Agent_v1_0.ipynb` (v1.0 버전)
* `Step2_조별과제_AI_강사_Agent_v2_0.ipynb` (v2.0 버전)

---

## 🖼 발표 자료 (Presentation)

[![발표자료 확인](https://github.com/Kim-geun-woo/AI-Instructor--Agent--Development/blob/main/images/ppt.png)](https://github.com/Kim-geun-woo/AI-Instructor--Agent--Development/blob/main/docs/AI%20%EB%AF%B8%ED%94%84%203%EC%B0%A8_05%EB%B0%98_09%EC%A1%B0.pdf)
> 📌 썸네일을 클릭하면 발표자료 PDF를 바로 확인할 수 있습니다.

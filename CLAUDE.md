# VLM-AOI Project — CLAUDE.md

## 프로젝트 개요
기구물 비전 검사 시스템. 두 버전을 병행 개발·동기화한다.

| 버전 | 파일 | 상태 |
|------|------|------|
| Standalone HTML | `기구물 비전 검사_vN.html` | v2 진행중 |
| Full-stack | `fullstack/` (FastAPI + React) | 설계 완료, 구현 대기 |
| 원본 백업 | `기구물 비전 검사_base.html` | 읽기 전용 |

---

## 개발 원칙 — Patch-Based Development

> **상세 규칙은 `PATCH_DEV_SKILL.md` 참조**

핵심 요약:
1. **전체 파일 재작성 금지** — Edit 툴로 최소 패치만 적용
2. **Read → Grep → Edit → Grep** 순서 준수
3. **패치 식별자** `P[기능번호]-[알파벳]` 사용 (예: P01-E)
4. **두 버전 항상 동기화** — HTML 패치 후 Full-stack 반영
5. **채널화 패턴 준수**: `function foo(ch) { ch = ch || ach(); ... }`

---

## 기술 스택

### Standalone HTML
- OpenCV.js 4.8.0 (WASM) — 실시간 비전 처리
- TensorFlow.js + MobileNet v2 — DL 크로스 검증
- Chart.js — 양품률 차트
- SheetJS (XLSX) — 검사이력 내보내기
- File System Access API — 폴더 저장

### Full-stack
- Backend: FastAPI + Python OpenCV
- Frontend: React + TailwindCSS
- WebSocket: 실시간 프레임 스트리밍

---

## 다중 채널 아키텍처 (v2 핵심)

```javascript
const MAX_CH = 4;
let channels = [];          // createChannel(id) 팩토리로 생성
let activeChannelId = 0;
function ach() { return channels[activeChannelId]; }
```

채널 상태 오브젝트 주요 속성:
```
ch.video, ch.canvas, ch.ctx     — DOM 참조
ch.isInspecting, ch.isFrozen    — 검사 상태
ch.masterMats[]                 — 기준 이미지 목록
ch.masterFileRegistry           — 바코드→이미지 레지스트리
ch.stats                        — {total, ok, ng, history[]}
ch.roi, ch.roiMode, ch.roiMask  — ROI 설정
ch.flipH, ch.cameraAngle        — 카메라 설정
ch.okSaveDirHandle              — OK 저장 폴더 핸들
```

공유 자원:
```
globalDlModel, globalIsDlReady  — MobileNet 모델 (채널 공유)
chartInstance                   — Chart.js 인스턴스
_modalChannel                   — 저장 모달 활성 채널 추적
```

---

## 개선 로드맵

| 우선순위 | 기능 | 버전 | 상태 |
|---------|------|------|------|
| 1 | 다중 카메라 (4채널) | 양쪽 | v2 진행중 |
| 2 | AI 모델 고도화 | 양쪽 | 대기 |
| 3 | 결과 리포트 | 양쪽 | 대기 |
| 4 | 설정 저장 (DB/API) | Full-stack만 | 대기 |

---

## 현재 패치 상태 (2026-06-15)

### P01 — 다중 카메라
- [x] P01-A: base 복사 → v2 생성
- [x] P01-B: CSS — 채널 탭 + 그리드 뷰
- [x] P01-C: HTML — 멀티 video/canvas
- [x] P01-D: JS — 채널 상태 팩토리
- [x] P01-E: JS — 모든 함수 채널화 완료
  - processVideo, performInspection, updateResult, updateStatsUI
  - logHistory, promptOkBarcode, savePassImage
  - drawTextUpright, drawOverlay, toggleInspectMode
  - setMasterImage, openSaveMasterModal, saveMasterImageFile
  - renderRegistryList, saveSettings, loadSettings
  - setupEventListeners, processBarcodeInput, extractSmartContour
  - bindCanvasEvents, unbindCanvasEvents (신규 추가)
- [ ] P01-F: 그리드 뷰 렌더링 완성 (renderGridView)
- [ ] Full-stack: FastAPI 다중 채널 API
- [ ] Full-stack: React 다중 카메라 UI

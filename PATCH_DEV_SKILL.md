# Patch-Based Development Skill (VLM-AOI 프로젝트 공용)

## Purpose
코드 확장/수정 시 전체 파일 재작성 대신 **최소 단위 패치**만 적용한다.
- 토큰 소모 최소화
- 할루시네이션 방지
- 맥락 연속성 유지

---

## Core Rules

### 1. 절대 금지
- 전체 파일 재작성 / Write 툴로 덮어쓰기 금지
- Read 없이 Edit 적용 금지
- 한 Edit 호출에 500줄 초과 금지

### 2. 패치 식별자
```
P[기능번호]-[A~Z]   예: P01-B = 기능#1 두 번째 패치(CSS)
```
| 접미사 | 카테고리 |
|--------|---------|
| A | 파일 생성/복사 (base → vN) |
| B | CSS 패치 |
| C | HTML 구조 패치 |
| D | JS 상태/팩토리 패치 |
| E | JS 함수 채널화 패치 |
| F | JS 이벤트/바인딩 패치 |

### 3. Edit 원칙
```
old_string: 파일 내 유일한 문자열 (전후 컨텍스트 최소 2줄 포함)
new_string: 변경 내용만
replace_all: 변수명 전체 교체 시만 true
```

### 4. 패치 워크플로
```
1. Read  → 대상 구역 확인 (라인 범위 지정)
2. Grep  → old_string 유일성 확인
3. Edit  → 패치 적용 (P번호 명시)
4. Grep  → 잔존 old 패턴 검증
```

### 5. 두 버전 동기화
- Standalone HTML (`기구물 비전 검사_vN.html`) + Full-stack 항상 병행
- 기능 완성 후 동기화 체크 수행
- 동기화 불가 시 즉시 사용자 보고 + 대안 제시

### 6. 채널화 패턴 (이 프로젝트)
```javascript
// BEFORE (단일 state)
function foo() { state.xxx; outCanvas.yyy; video.zzz; }

// AFTER (ch 파라미터)
function foo(ch) {
    ch = ch || ach();
    ch.xxx; ch.canvas.yyy; ch.video.zzz;
    if (ch.id === activeChannelId) { /* DOM 업데이트만 */ }
}
```

### 7. Anti-patterns (금지)
- `// ... 기존 코드 유지 ...` 주석으로 코드 생략
- 패치 없이 설명만 제공
- Read 없이 함수 전체 재작성
- 500줄 초과 단일 Edit

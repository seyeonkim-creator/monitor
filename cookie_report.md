# [보고서] 쿠키 동의 배너 및 GTM 연동 데이터 흐름 (보안팀 제출용)

본 문서는 웹사이트 내 쿠키 동의 배너 도입에 따른 데이터 수집 로직과 GTM(Google Tag Manager)을 통한 개인정보 통제 매커니즘을 설명합니다.

---

## 1. 개요 (Executive Summary)
- **목적**: 사용자의 명시적 동의(Opt-in) 후에만 마케팅용 개인정보 및 쿠키 수집 허용.
- **핵심 원칙**: **Default Deny(기본 거부)**. 동의 전까지는 모든 매체 태그의 식별 정보 수집을 원천 차단함.
- **관리 도구**: GTM '동의 모드(Consent Mode)'를 통한 중앙 집중식 제어.

---

## 2. 데이터 흐름 시각화 (Data Flow Diagram)

> **참고**: 아래는 Mermaid 다이어그램 코드입니다. 마크다운 뷰어(Notion, VS Code 등)에서 자동으로 차트로 변환됩니다.

```mermaid
graph TD
    A[사용자 웹사이트 접속] --> B{GTM 초기화<br/>Default: Denied}
    B --> C[쿠키 동의 배너 노출]

    %% 동의 거부 시
    C -->|동의 거부 / 무시| D[Consent Update: Denied유지]
    D --> F{매체 태그 작동 확인}
    F -->|Meta / Kakao 등| G[<b>수집 완전 차단</b><br/>데이터 전송 0%]
    F -->|Google 태그| H[<b>익명 핑 전송</b><br/>쿠키 미사용/식별불가]

    %% 동의 허용 시
    C -->|동의 클릭| E[Consent Update: Granted변경]
    E --> I[dataLayer.push<br/>'consent_confirmed' 발생]
    I --> J[GTM 트리거 작동]
    J --> K[매체 태그 정식 실행]
    K --> L[<b>데이터 수집 완료</b><br/>개인식별자/쿠키 전송]

    style G fill:#f96,stroke:#333
    style L fill:#9f9,stroke:#333

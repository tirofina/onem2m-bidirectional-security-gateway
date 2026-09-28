# 제안서용 시장·경쟁·재무 근거 메모

확인일: **2026-09-22**. 공식 제품 문서·공식 실적 발표만 사용했다. 아래 8개 항목은 경쟁 기능 5개와 인접 시장 실적 3개다. 제품 기능은 공급업체가 공개한 기능 설명이며, 실증 성능이나 우리 제품의 우위를 증명하지 않는다.

## 제안서에 먼저 반영할 판단

**IoT 통신을 감시하고 이상 행위를 차단하는 제품은 이미 있다.** 차별화 가설은 “oneM2M의 리소스·요청 의미, 인증된 기기 정체성, 장치 상태와 데이터 신뢰도를 함께 사용해 양방향 정책을 실행하고 Mobius4 운영에 쉽게 결합한다”로 좁혀야 한다. 아래 공개 자료에서 해당 조합의 완성형 제품을 확인하지 못한 것은 시장 공백이나 신규성의 증명이 아니다. oneM2M 전용 제품의 고객 수, 구매 의사, 설치 기반과 예산은 본 조사로 검증되지 않았다.

“일반 IPS는 내용을 이해하지 못한다”, “기존 제품은 탐지만 한다”, “명령 실행 전 차단은 최초다”라는 표현은 사용하지 않는다. 경쟁사도 프로토콜 파라미터 차단, 행동 기반 격리, 방화벽 연동을 제공한다. 우리 목표는 기존 방어 위에 추가되는 가치를 비교 실험으로 입증하는 것이다.

## 1. Fortinet FortiGate / FortiGuard OT Security

- 대상: 산업·OT 네트워크. 가장 직접적인 기능 비교 대상이다.
- 공식 백서는 프로토콜 페이로드 DPI로 기능·값을 식별하고 방화벽 정책에서 허용·거부할 수 있다고 설명한다. Modbus 읽기를 허용하고 나머지 기능을 막는 예를 제시한다. OT 전용 IPS와 가상 패치도 제공한다.
- 집행 방식: **트래픽 경로의 방화벽이 직접 차단**. 따라서 “프로토콜을 이해한 위험 명령 차단” 자체를 독창성으로 주장할 수 없다.
- 비교 검증점: oneM2M 리소스 트리·AE/CSE 정체성, 센서 신뢰도와 장치 상태의 연계, Mobius4 결합 편의성을 별도 시험한다. 미공개 기능은 “없음” 대신 “본 조사에서 확인하지 못함”으로 표시한다.
- 출처: [공식 OT 백서, 2022, 특히 pp.4·6·8](https://www.fortinet.com/content/dam/fortinet/assets/white-papers/wp-advanced-threat-protection-industrial-control-systems-ot.pdf), [현행 FortiGuard OT 서비스 설명](https://www.fortinet.com/support/support-services/fortiguard-security-subscriptions/industrial-security).

## 2. Nozomi Networks Guardian

- 대상: OT·IoT 자산 가시성, 행동 학습과 위협 탐지.
- Guardian 센서는 수동적으로 트래픽을 관찰한다. 공식 관리 문서는 외부 방화벽에 정책을 삽입해 학습되지 않은 노드·연결 또는 특정 경보에 따른 연결을 차단하고, 일부 연동에서 세션 종료도 지원한다고 설명한다.
- 집행 방식: **수동 탐지 + 연동된 방화벽의 차단**. “탐지 전용이라 차단 불가”로 단순화하면 틀린 비교다. 요청마다 물리적 실행 전에 동기적으로 판정하는 우리 목표 구조와 탐지 후 정책 반영 구조의 지연·보장 범위를 구분한다.
- 출처: [Guardian 공식 제품 설명](https://www.nozominetworks.com/platform/guardian), [Firewall integration 공식 관리자 문서](https://technicaldocs.nozominetworks.com/guardian/latest/products/n2os/topics/administration/settings/firewall-integration/c_n2os_admin_settings_firewall-integration.html).

## 3. AWS IoT Device Defender

- 대상: AWS IoT 기기 설정 감사, 지표 기반 규칙·ML 이상 탐지, 경보 및 완화 조치.
- 공식 문서는 알림과 함께 thing group 변경, 정책 버전 교체, 인증서 변경 등의 대응을 설명한다. 메트릭·경보 기반 서비스이므로 모든 장치 명령을 실행 전에 동기적으로 판정하는 게이트와 동일하지 않다.
- **2026-08-31부터 Detect는 신규 고객에게 제공되지 않고 maintenance mode로 전환됐다. 기존 고객은 계속 사용 가능하며 Audit는 영향을 받지 않는다.** 신규 도입 대안으로 소개할 때 이 조건을 반드시 함께 쓴다.
- 출처: [공식 서비스 개요](https://docs.aws.amazon.com/iot-device-defender/latest/devguide/what-is-device-defender.html), [Detect 가용성 변경 공식 공지](https://docs.aws.amazon.com/iot-device-defender/latest/devguide/dd-detect-availability-change.html).

## 4. Firewalla / Device Active Protect

- 대상: 가정·소규모 사무실 네트워크 보안. 인라인 방화벽·IDS/IPS와 악성 목적지 차단을 제공한다.
- Device Active Protect는 장치 통신을 학습해 필요한 목적지만 허용하는 기능이다. 문서상 Gold 계열, Router 또는 Bridge 모드 등 조건이 있으며 모든 장치가 대상은 아니다. LAN 기기 격리는 Firewalla AP7 조건을 명시한다.
- 집행 방식: **통신 경로에서 직접 차단**, 행동 학습·허용 목록 중심. 따라서 일반인용 보안 허브나 IoT 발신 통신 제어만으로는 차별화하기 어렵다.
- 출처: [차단 방식 공식 도움말](https://help.firewalla.com/hc/en-us/articles/360007210473-Where-Can-Firewalla-Block), [Device Active Protect 공식 도움말](https://help.firewalla.com/hc/en-us/articles/44061066094867-Device-Active-Protect-Dynamic-Microsegmentation-Block-everything-and-allow-only-what-s-needed).

## 5. CUJO AI Sentry

- 대상: 통신사업자가 가정용 라우터를 통해 제공하는 네트워크 보안 서비스. 위협 탐지와 차단을 제공하며 B2B2C 유통의 사례다.
- 공식 2024 보고서는 가정 네트워크 장치 보호를 설명한다. 과거 기술 백서는 Device Sentry가 패킷 헤더·통신 상대의 행동을 분석하며 DPI를 하지 않는다고 설명한다. **과거 문서의 구현 설명을 현행 제품 전체의 한계로 단정하지 않는다.**
- 현행 홈페이지의 “6천만 가구 이상 배포”는 공급업체의 배포 규모 주장이다. 유료 가입자 수, 당해 연도 판매량 또는 제품 이익률로 환산할 수 없다.
- 출처: [공식 홈페이지](https://cujo.com/), [2024 공식 보안 보고서](https://cujo.com/wp-content/uploads/2024/11/CUJO-AI_Cybersecurity-Report_2024.pdf), [기술 백서, 저작권 2020, p.8](https://cujo.com/wp-content/uploads/2021/08/CUJO-AI_Applying-Artificial-Intelligence.pdf).

## 6. Verisure — FY2025, 보안 관제 구독 인접 시장

- 회계연도: 2025년 1~12월. 매출 **€3,745.4m**, 조정 EBIT **€952.9m**, 조정 EBIT 마진 **25.4%**.
- 계산: 952.9 ÷ 3,745.4 × 100 = **25.44%**. 회사가 공시한 조정 지표이며 순이익률이 아니다.
- 2025년 말 보호 고객은 **약 620만**, 2025년 4분기 신규 설치 **22.4만 건**. 전자는 연말 고객 기반, 후자는 분기 설치 건수다. 연간 판매량·기기 수와 혼용하지 않는다.
- 적용 범위: 전문 인력의 감시·대응을 포함한 기업 전체 서비스. oneM2M 사이버보안 허브 판매량·마진은 이 공시에서 분리 확인되지 않는다.
- 출처: [FY2025 공식 실적 발표, 2026-02-12, 상세 표 기준](https://www.verisure.com/press-releases/regulatory/verisure-reports-q4-and-full-year-2025-results), [2025 연차보고서 CEO 설명—고객 수](https://www.verisure.com/annual-reports/2025/comments-from-the-ceo).
- 참고: 발표문 제목 요약의 성장률은 고정환율 기준이 섞여 있다. 제안서에는 불필요한 성장률을 생략하거나 보고 매출 성장률 **9.9%**와 고정환율 **10.3%**를 구분한다.

## 7. Allegion — FY2025, 출입 보안 하드웨어·솔루션 인접 시장

- 회계연도: 2025년 1~12월. 매출 **US$4,067.3m**, 영업이익 **US$859.5m**, GAAP 영업이익률 **21.1%**.
- 계산: 859.5 ÷ 4,067.3 × 100 = **21.13%**. 조정 영업이익률 23.2%와 혼용하지 않는다.
- 연간 매출은 전년 대비 7.8% 증가했으나, 확인한 실적 발표에는 개별 스마트 도어락의 출하 수량·제품별 이익률이 없다.
- 적용 범위: 자물쇠, 도어 하드웨어, 출입 통제 등 전사 실적이다. 당사 보안 허브의 예상 마진이나 시장 규모로 사용할 수 없다.
- 출처: [FY2025 공식 실적 발표, 2026-02-17](https://www.allegion.com/corp/en/news/year/2026/2025-results.html).

## 8. Resideo — FY2025, 센싱·제어·유통 인접 시장

- 회계연도: 2025년 1~12월. 당시 전사 매출 **US$7,472m**, 매출총이익 **US$2,196m**, 영업이익 **US$607m**.
- 매출총이익률: 2,196 ÷ 7,472 × 100 = **29.39% → 29.4%**. 영업이익률(본 조사 계산): 607 ÷ 7,472 × 100 = **8.12%**.
- 조정 EBITDA **US$833m** / 매출 = **11.15%**. GAAP 순손실 **US$527m**이며 배상 약정 종료 관련 비용이 영향을 줬다. 총이익률 29.4%를 순이익률로 표현하면 안 된다.
- 적용 범위: Products & Solutions와 ADI 유통을 포함한 FY2025 전사 기준이다. 개별 보일러 보안 키트 판매량·마진은 해당 공시에서 분리 확인되지 않는다. 현행 사업구조와 동일하다는 가정도 하지 않는다.
- 출처: [FY2025 공식 실적 발표, 2026-02-24, Financial Highlights 및 Table 1/2](https://investor.resideo.com/news/news-details/2026/Resideo-Announces-Fourth-Quarter-and-Full-Year-2025-Financial-Results-and-Initiates-2026-Outlook/).

## 표에 옮길 때의 원칙

| 기업/회계연도 | 매출 | 수익성 지표 | 판매·규모 지표의 의미 |
|---|---:|---:|---|
| Verisure FY2025 | €3,745.4m | 조정 EBIT 마진 25.4% | 연말 약 620만 고객; Q4 신규 설치 22.4만 |
| Allegion FY2025 | US$4,067.3m | GAAP 영업이익률 21.1% | 개별 스마트 도어락 수량·마진 미확인 |
| Resideo FY2025 | US$7,472m | 매출총이익률 29.4%; 영업이익률 8.12%(계산) | 개별 보안 키트 수량·마진 미확인 |

이 표는 인접 기업의 사업 규모를 보여준다. **서로 다른 이익률을 순위로 비교하지 않으며, 우리 제품 매출·판매량·마진 예측의 근거로 사용하지 않는다.** “자료에서 미확인”은 세상에 공개된 수치가 전혀 없다는 뜻이 아니다.

## 시장성 검증 계획으로 권하는 문구(실적이 아닌 제안)

초기 고객은 oneM2M/Mobius 운영기관, 구축 SI, 시설 IoT 운영사를 우선 가설로 둔다. 10개 이상의 운영 현장 인터뷰에서 최근 장애·오조작·보안사고, 기존 대응 비용, 필요한 연동 방식, 구매 예산과 승인자를 확인한다. 그중 2~3곳에서 유료 또는 비용분담 실증 의사를 확인한 뒤 출시 범위를 결정한다. 데모 키트의 반응이나 기술 탐지율만으로 구매 의사가 입증됐다고 판단하지 않는다.

기술 비교는 기본 oneM2M 보안, 기본 보안+IPS, 기본 보안+IPS+당사 정책의 세 조건으로 수행한다. 새 정책 때문에 줄어든 위험 명령, 오차단, 추가 지연과 정책 운영 시간을 함께 측정한다. 추가 효용이 작거나 통합비용이 높으면 독립 하드웨어 판매보다 Mobius 운영 도구·정책 모듈 공급 모델을 재검토한다.

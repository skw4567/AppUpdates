# the PLUS Piping (플랜트 배관 용접 관리 시스템)

**the PLUS Piping**은 플랜트 건설 현장의 배관 도면 분석 및 용접 조인트 데이터를 체계적으로 관리하기 위한 통합 데스크탑 솔루션입니다.

---

## 🚀 주요 기능 (Key Features)

* **도면 분석 및 데이터 통합**: 도면 내 용접 포인트(Joint) 자동 추출 및 매칭.
* **Zero-Config 네트워크**: UDP Beacon 기술을 이용한 작업자 PC-서버 PC 자동 연결.
* **지능형 업데이트**: GitHub API를 활용한 실시간 증분 업데이트 시스템.
* **커스텀 리포트**: `.tppr` 포맷을 활용한 동적 리포트 생성 및 데이터 그룹화.

---

## 📦 설치 및 배포 유형 (Installation Modes)

설치 시 사용자 환경에 맞는 모드를 선택할 수 있습니다:

1.  **단독 사용자 모드 (Standalone)**: 로컬 PC에 SQLite를 사용하여 개인 작업을 수행합니다.
2.  **다중 사용자 [서버] (Server)**: 메인 PC에 PostgreSQL 서버를 서비스로 등록하여 데이터를 중앙 관리합니다.
3.  **다중 사용자 [클라이언트] (Client)**: 서버 PC에 접속하여 데이터를 공유하며 협업합니다.

---

## 🛠️ 기술 스택 (Tech Stack)

* **Language**: Python 3.11+
* **GUI Framework**: PySide6 (Qt for Python)
* **Database**: SQLite (Local), PostgreSQL (Server)
* **Build Tool**: Nuitka (Standalone Executable)
* **Installer**: NSIS (Nullsoft Scriptable Install System)
* **Deployment**: GitHub Releases (Update Server)

---

## 💻 개발 환경 설정 (Getting Started)

### 1. 요구 사항
* Windows 10/11 (64-bit)
* PostgreSQL 15+ (서버 모드 사용 시)


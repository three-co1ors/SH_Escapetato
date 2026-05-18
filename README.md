# 🌱 왓더팜 (What The Fxxm!) — 개인 기여 정리

**왓더팜**은 플레이어가 작은 농장의 주인이 되어  
작물을 심고, 수확하고, 연구하며 농장을 성장시키는  
**2D 모바일 방치형 농장 게임**입니다.

퀘스트를 통해 콘텐츠를 해금하고,  
다양한 아이템과 테마로 나만의 농장을 완성할 수 있습니다.

<br>

![Unity](https://img.shields.io/badge/Unity-000000?style=flat-square&logo=unity&logoColor=white)
![C#](https://img.shields.io/badge/C%23-239120?style=flat-square&logo=csharp&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat-square&logo=firebase&logoColor=black)
![Android](https://img.shields.io/badge/Android-3DDC84?style=flat-square&logo=android&logoColor=white)

<br>

## 📌 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 게임명 | 왓더팜 (What The Fxxm!) |
| 장르 | 2D 모바일 방치형 농장 게임 |
| 플랫폼 | Android |
| 기술 스택 | Unity, C#, Firebase (Auth, Realtime Database) |
| 소속 | 숙명여자대학교 프로그래밍 중앙동아리 SOLUX 2025-2학기 게임 프로젝트 |
| 팀 구성 | 5인 (기획 1, 디자인 1, 개발 3) |

<br>

## 👥 역할 분담

- 기획: 컴과23 박지윤 (개발 겸업)
- 디자인: IT21 허희윤
- 개발: 컴과22 장은빈, 컴과22 정채연, 컴과23 박성하

<br>

## 👩‍💻 박성하(three-co1ors) 담당 파트

### 1. 프로젝트 초기 세팅
- Unity 프로젝트 생성 및 폴더 구조 설계
- Android 빌드 환경 세팅

### 2. 인벤토리 / 도감 / 연구소 / 상점 시스템

팀원들이 공통으로 사용하는 핵심 시스템을 설계하고 구현했습니다.

- `InventoryManager` — 싱글톤 기반 아이템 추가/제거/수량 조회 (`AddItem`, `RemoveItem`, `GetItemCount`)
- `도감` — 작물 수확/진화 시 `GameProgressionManager.UnlockItem()` 호출로 자동 등록
- `연구소` — 작물 + 포션 조합으로 작물 진화, 진화 이미지 추가
- `상점` — 씨앗/아이템 구매, 판매 기능 및 판매 가격 설정
- `PoingManager` — 재화 관리 (`AddPoing`, `DecreasePoing`, `HasEnoughPoing`)

### 3. UIManager 설계 (팝업 관리 아키텍처)

여러 시스템에서 동시에 팝업을 열 때 충돌이 발생하는 문제를 방지하기 위해 **중앙 팝업 관리 아키텍처**를 설계했습니다.

- 싱글톤 `UIManager`에서 모든 팝업의 열기/닫기를 중앙 통제
- `CloseAllPopups()` 이후 해당 팝업만 열도록 하여 팝업 중복 방지
- 팀 내 공통 가이드라인 문서 작성 → 팀원들이 `SetActive` 직접 호출 없이 UIManager API만 사용하도록 규칙화

```csharp
// UIManager 사용 예시
UIManager.Instance.OpenQuestPopup();
UIManager.Instance.ShowAlertPopup("포잉이 부족합니다!");
UIManager.Instance.ShowItemAcquiredPopup(itemData);
```

**[사용 관련 노션]**
https://cake-beast-9f8.notion.site/2b1ff9ca48fb80469098fd3a5a647079?source=copy_link


### 4. 세이브 시스템 (Firebase Realtime Database)

- Firebase Realtime Database 기반 세이브/로드 구현
- QuestManager 진행 상황 DB 저장


### 5. Firebase Auth + Google 로그인 설정

- Firebase 프로젝트 생성 및 Android 앱 등록
- `google-services.json` 발급 및 Unity 연결
- Firebase Auth SDK, Google Sign-In 플러그인 설치 및 세팅
- SHA-1 생성 및 Firebase 콘솔 등록
- 팀 내 Firebase 로그인 설정 가이드 문서 작성

### 6. BGM / 효과음

- `SoundManager` 작성

### 7. 씬 통합 및 퀘스트 통합

- 독립적으로 개발된 인벤토리, 연구소, 상점 등 각 시스템을 메인 씬에 통합
- 퀘스트 시스템 메인 씬 통합

<br>

## 🔧 트러블슈팅

### 1. 퀘스트창 열고 닫으면 포잉이 사라지는 버그

**문제:** 퀘스트 팝업을 열었다 닫을 때마다 포잉(재화)이 0으로 초기화되는 현상 발생.

**원인:** `UIManager.cs`에서 `CloseAllPopups()` 호출 시 연결된 오브젝트가 `PoingManager` UI를 포함하고 있어, 닫힐 때 포잉 표시 오브젝트까지 비활성화되고 재활성화 시 초기값으로 렌더링되고 있었음.

**해결:** `CloseAllPopups()`가 팝업 오브젝트만 정확히 끄도록 대상을 명시적으로 분리하여 수정.

---

### 2. 세이브 Race Condition — DB 로딩 전에 저장이 실행되는 문제

**문제:** 게임 시작 시 QuestManager의 진행 상황이 항상 초기값으로 덮어씌워지는 현상.

**원인:** `Start()`에서 Firebase DB 로딩이 완료되기 전에 `SaveToDB()`가 먼저 실행되어, 아직 로드되지 않은 빈 데이터(초기화 상태)로 DB를 덮어쓰고 있었음.

**해결:** `QuestManager`에 `isLoaded` 플래그를 추가하여 DB 로딩 완료 이후에만 저장이 실행되도록 Guard 처리.

```csharp
private bool isLoaded = false;

void SaveToDB() {
    if (!isLoaded) return; // 로딩 전에 저장 차단
    // 저장 로직
}
```

---

### 3. 일일 퀘스트 껐다 켜면 리셋되는 문제

**문제:** 앱을 재시작하면 일일 퀘스트 진행 상황이 초기화되는 현상.

**원인 1:** `ConfigureDailyQuests()` (일일 퀘스트 생성) 직후 `SaveToDB()`를 호출하지 않아, 생성된 퀘스트가 DB에 저장되지 않고 매번 새로 생성되고 있었음.

**원인 2:** 로드 시점에 `TargetCount`(목표 수치)가 0으로 날아가, UI가 `0/0` 으로 깨져 보이는 현상 동반.

**해결:**
- 퀘스트 생성 직후 `SaveToDB()` 호출 추가
- 로드 시점에 `TargetCount`가 0이면 기본값으로 복구하는 예외 처리 추가

---

### 4. 데이터 구조 불일치 — 배열 vs 정수

**문제:** 퀘스트 진행 카운트가 DB에 저장되지 않는 현상.

**원인:** 인게임 코드는 `currentCounts` (배열)로 퀘스트 진행 수를 관리하는데, DB 저장 구조는 `currentCount` (단일 정수)로 설계되어 있어 저장/로드 시 값이 매핑되지 않았음.

**해결:** 저장 시점에 배열 → 정수, 로드 시점에 정수 → 배열로 강제 동기화하는 변환 코드 추가.

---

### 5. 메인/일일 퀘스트 Key 충돌

**문제:** 특정 메인 퀘스트와 일일 퀘스트의 진행 상황이 서로 섞이는 현상.

**원인:** 메인 퀘스트와 일일 퀘스트의 DB Key(ID)가 둘 다 `1`부터 시작하도록 설계되어 있어, Firebase DB에서 동일한 키로 저장/덮어쓰기가 발생.

**해결:** 메인 퀘스트와 일일 퀘스트의 Key 네임스페이스를 분리하여 충돌 방지.

---

### 6. Google Sign-In 플러그인 Unity 2022 버전 충돌

**문제:** Google Sign-In Unity 플러그인 임포트 후 컴파일 에러 발생.

**원인:** 플러그인 내 레거시 DLL(`Unity.Compat.dll`, `Unity.Tasks.dll`)이 Unity 2022 내장 라이브러리와 충돌.

**해결:** 충돌 DLL 2개를 수동으로 제거 후 Google Version Handler에서 obsolete 파일 정리(Apply) 수행.

```
제거 파일:
- Unity.Compat.dll ❌
- Unity.Tasks.dll ❌
```

<br>

## 🔗 링크

- [시연 영상](https://drive.google.com/file/d/1-rtY5uWgW6IE5pb5Hn5tVtAR6nsgg_3L/view?usp=sharing)
- [게임 다운로드](https://drive.google.com/file/d/1dP9Q-V9O2qbAE9nGyTsHmJEQQpLCcO6f/view?usp=sharing)
- [팀 레포지토리](https://github.com/Escapetato/developer)

<br>

## 🛠️ 개발 환경

- Engine: Unity
- Platform: Mobile (Android)
- Language: C#
- Version Control: Git
- Collaboration: Notion

<br>

## ✨ 주요 특징

- 🌾 방치형 재배 시스템  
- 📜 메인 / 서브 / 일일 퀘스트 구성  
- 🧪 에너지와의 조합으로 작물 진화
- 📚 도감에서 수확/진화한 작물 모아보기
- 🎨 농장 꾸미기 테마 변경

<br>

## 🎮 시스템 및 컨텐츠 소개

### 🌾 밭

<img width="3840" height="2160" alt="--전체구상도" src="https://github.com/user-attachments/assets/8365980c-5ebd-4c62-b9b4-5cc56d0c5b3b" />

플레이어는 밭에서 작물을 재배할 때, 특정 단계들로 이루어진 루프를 반복하게 됩니다.
1. 씨앗 구매
2. 씨앗 심기
3. 작물 별 재배시간 대기
4. 비료 구매 및 사용
5. 알맞은 도구로 수확
6. 작물 판매 또는 작물 진화

<br>

### 📜 퀘스트
- 메인퀘스트: 게임의 전체적인 흐름을 유도하는 스토리형 퀘스트
  - 보상: 땅 확장, 수확도구 획득 (각 3단계씩, 총 6단계)
- 서브퀘스트: 작물 씨앗 해금을 유도하는 퀘스트
  - 보상: 작물 씨앗 해금 + 소액의 포잉
- 일일퀘스트: 출석포인트 개념의 간단한 반복작업 퀘스트
  - 보상: 포잉, 비료, 포션 등 퀘스트 난이도에 따라 다름

메인퀘스트 클리어 후에는 서브 / 일일 퀘스트 중심으로 플레이가 진행됩니다.

<br>

### 🧪 연구실
수확한 작물과 상점에서 구매한 포션을 조합하여 다양한 방식으로 작물을 진화시킬 수 있습니다.

<img width="674" height="301" alt="스크린샷 2026-01-24 21 03 09" src="https://github.com/user-attachments/assets/bb6ec480-421a-4a53-8e44-52d6f8209d97" />

<img width="692" height="192" alt="스크린샷 2026-01-24 21 05 54" src="https://github.com/user-attachments/assets/bcf629e6-e4f7-4d3c-9184-a4a6525b6994" />


## 🎥 시연영상
https://drive.google.com/file/d/1-rtY5uWgW6IE5pb5Hn5tVtAR6nsgg_3L/view?usp=sharing


👉 [게임 해 보러 가기](https://drive.google.com/file/d/1dP9Q-V9O2qbAE9nGyTsHmJEQQpLCcO6f/view?usp=sharing)

(Google Drive 실행파일로 연결됩니다. Google Drive에서 다운로드되는 실행 파일은 보안 경고가 표시될 수 있습니다.)

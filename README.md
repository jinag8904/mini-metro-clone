# 🚇 Subway Line Manager — Toy Project 0 (Team 7)

> 승객이 폭발하기 전에 지하철 노선을 그려 도시를 굴러가게 만드는 **실시간 노선 경영 게임**입니다.
> Unity 6로 제작한 3인 팀 토이 프로젝트이며, *Mini Metro*의 게임 디자인을 학습 목적으로 재해석해 직접 구현했습니다.

<div align="center">

`Unity 6000.3.11f1` · `C#` · `2D` · `3인 팀` · `2026.04.29 ~ 2026.05.07`

</div>

---

## 🎮 어떤 게임인가요?

지도 위에 역이 하나둘 생겨나고, 각 역에는 **가고 싶은 목적지 모양**을 머리에 띄운 승객이 끊임없이 도착합니다.
플레이어는 마우스로 역과 역을 이어 **노선을 그리고**, 그 위에 **열차를 배치**해 승객을 목적지까지 실어 나릅니다.

- 승객은 자기가 원하는 **모양(●, ■, ▲)의 역**에 도착하면 내립니다.
- 역마다 수용 인원이 정해져 있고, **초과되면 초읽기 타이머가 돌아갑니다.**
- 타이머가 0이 되면 그 역은 폐쇄되고 **게임 오버** — 그때까지 몇 명을 실어 날랐는지가 곧 점수입니다.

역은 시간이 지날수록 계속 늘어나기 때문에, "지금 이 노선을 어디로 연장할 것인가"를 매 순간 다시 판단해야 하는
**공간 설계 + 자원 배분 + 타임 어택**이 한 번에 걸린 구조입니다.

## 🕹 조작

| 조작 | 동작 |
|---|---|
| 역 클릭 후 드래그 | 새 노선 만들기 |
| 노선 끝 핸들 드래그 | 기존 노선 연장 / 단축 |
| 노선 중간 클릭 후 드래그 | 노선 중간에 역 끼워 넣기 (경로 편집) |
| 하단 노선 버튼 클릭 | 해당 노선 전체 삭제 (사용한 자산은 반환) |
| 자산 아이콘 → 노선으로 드래그 | 열차 · 객차 · 고속열차 배치 |
| 자산 아이콘 → 역으로 드래그 | 해당 역을 환승역으로 승격 |
| ⏸ / ▶ / ⏩ | 일시정지 · 재생 · 2배속 |

## ⚙️ 핵심 시스템

### 노선 (Line)
- 최대 **7개 노선**, 노선당 최대 **4대의 열차**. 시작은 3개 노선으로 제한됩니다.
- 두 역을 이으면 *Mini Metro* 특유의 **45° 꺾임 경로**가 자동 생성됩니다 (`Line.GetBendPoint`).
- 첫 역과 마지막 역을 이으면 **순환선**이 되어 열차가 한 방향으로 계속 돕니다.
- 노선을 편집하면 **주행 중인 열차는 즉시 튕기지 않고**, 다음 정차 타이밍에 새 경로를 반영합니다 (경로 변경 예약 플래그).

### 승객 & 길찾기 (Passenger / Train)
- 승객은 자신이 탄 열차의 노선을 **BFS로 탐색**해 목적지 모양의 역에 닿을 수 있는지 판단합니다.
- 직통으로 갈 수 없으면 **환승 가능한 역을 찾아 그 역에서 하차** → 다른 노선으로 갈아탑니다.
- 갈아탈 노선이 없으면 열차에 남아 다음 기회를 기다립니다. 즉, **승객이 스스로 경로를 판단**합니다.

### 역 (Station)
- 원 · 사각 · 삼각 세 종류가 랜덤 위치·랜덤 간격으로 계속 생성됩니다 (첫 시작은 모양이 겹치지 않는 3개).
- 기본 수용 인원 **6명**, 초과 시 **30초 오버플로 타이머**가 링 UI로 표시됩니다.
- **환승역**으로 승격하면 수용 인원이 **10명**으로 늘고 크기가 커집니다.

### 주간 보상 (AssetManager)
- 게임 내 시간은 요일 단위로 흐르고, **매주 말 보상 선택창**이 열립니다.
- 기관차 1대를 무조건 받고, 추가로 **노선 / 환승역 / 객차 / 고속열차** 중 2개가 랜덤 제시되어 하나를 고릅니다.
- 노선을 삭제하면 거기에 투입했던 자산이 **다시 인벤토리로 반환**되어 재배치할 수 있습니다.

## 🧩 기술적으로 신경 쓴 부분

- **이벤트 기반 게임오버** — `Station.OnTimeOver` 이벤트를 `GameManager`가 구독. 역은 자기 상태만 알고, 카메라 연출·결과 UI·`timeScale` 정지는 매니저가 처리하도록 책임을 분리했습니다.
- **매니저 단위 책임 분리** — `LineManager`(노선), `TrainManager`(열차 주행 루프), `StationManager`(스폰), `PassengerManager`(목적지 추첨), `AssetManager`(자산·보상 경제), `MouseInput`(입력 상태 머신)로 나눠 기능별로 병렬 개발했습니다.
- **입력 상태 머신** — 마우스 입력을 `NewLine / ExtendLine / EditLine / NewTrain / HighTrain / InterchangeStation / Carriage` 모드로 관리해 드래그 종류마다 다른 규칙을 적용합니다.
- **열차 주행 곡선** — 웨이포인트 단위 이동에 가·감속 구간(`accelerationDist`, `decelerationDist`)을 넣어 역 진입/출발이 부드럽게 보이도록 처리했습니다. 객차는 위치 히스토리를 따라가는 방식으로 연결됩니다.
- **속도 배율과 UI 일시정지 공존** — 보상창이 뜰 때 `timeScale`을 0으로 만들되 **직전 배속을 저장했다가 복구**해서, 2배속으로 플레이하다 보상을 받아도 속도가 초기화되지 않습니다.

## 📁 프로젝트 구조

```
Assets/
├─ Scenes/            # MainTitleScene(타이틀) · Scene(인게임) · MenuScene · Sky(크레딧)
├─ Scripts/
│  ├─ GameManager.cs          # 게임 오버 처리, 날짜/점수 UI
│  ├─ AssetManager.cs         # 요일 진행, 주간 보상, 자산 인벤토리
│  ├─ MouseInput.cs           # 드래그 입력 상태 머신
│  ├─ CameraController.cs     # 게임오버 시 카메라 연출
│  ├─ GameSpeedController.cs  # 일시정지 / 배속
│  ├─ Line/                   # Line, LineManager, Handle (노선 생성·편집·삭제)
│  ├─ Station/                # Station, StationManager, StationTimerUI
│  ├─ TrainPassenger/         # Train, TrainManager, Passenger, PassengerManager, Score
│  ├─ Data/                   # StationType, PassengerState, Colors
│  └─ UI/                     # 타이틀 · 메뉴 · 게임오버 · 크레딧 UI
├─ Prefabs/ · Sprites/ · Resources/ · Fonts/
└─ ...
```

## ▶️ 실행 방법

```bash
git clone https://github.com/jinag8904/toy-project-0-team-7.git
```

1. **Unity 6000.3.11f1** 로 프로젝트를 엽니다 (Unity Hub → Add project from disk).
2. `Assets/Scenes/MainTitleScene.unity` 를 열고 재생합니다.
3. 빌드 순서는 `MainTitleScene → Scene → Sky → MenuScene` 으로 등록되어 있습니다.

## 👥 팀 & 개발 과정

| 이름 | GitHub |
|---|---|
| 김진아 | [@jinag8904](https://github.com/jinag8904) |
| 이하늘 | [@smflsmfqh](https://github.com/smflsmfqh) |
| 윤지수 | [@JISUYOON](https://github.com/yoonjisu1201) |

- **기간** · 2026.04.29 ~ 2026.05.07 (약 1.5주), 총 197 커밋
- **협업 방식** · `main` ← `dev` ← `feature/#이슈번호` 브랜치 전략, 모든 기능은 이슈 등록 → PR → 리뷰 후 머지
- **이슈 관리** · 버그 리포트 / 기능 요청 [이슈 템플릿](.github/ISSUE_TEMPLATE)을 만들어 사용
- **커밋 컨벤션** · `[Add]`, `[Correct]` 등 말머리로 변경 성격 구분

---

> 학습 목적으로 제작한 비상업 토이 프로젝트입니다. 게임 디자인은 Dinosaur Polo Club의 *Mini Metro* 에서 영감을 받았습니다.

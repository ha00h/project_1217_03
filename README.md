# 프로젝트3

플러터 기반 테트리스 게임을 구축하기 위한 기준 정보를 정리한 문서입니다. 향후 모든 구현은 아래 개요, 스택, 컨벤션, 요구사항을 준수합니다.

## 프로젝트 개요
- 플랫폼: Flutter 3.x / Dart 3.x
- 타깃: Android, iOS, Web (단일 코드베이스)
- 목표: 클래식 테트리스 룰을 기반으로 한 빠른 반응형 퍼즐 게임 제공

## 개발 스택
- Flutter (Material 3, CustomPainter, Game loop via `Ticker`/`Timer`)
- Riverpod 또는 Provider 기반 상태 관리 (선호: Riverpod)
- Freezed & json_serializable (데이터 모델 정의 시)
- very_good_analysis 혹은 flutter_lints (코드 품질)
- GitHub Actions (CI), Firebase App Distribution/Crashlytics (선택)

## 코딩 컨벤션
- 파일 및 클래스 명: PascalCase (예: `GameBoard.dart`, `TetrominoShape.dart`)
- 위젯/상수/Provider 네이밍도 PascalCase 유지, 상수는 `k` prefix 없이 의미 있는 이름 사용
- Widget 트리와 로직 분리: 렌더링 위젯 vs 상태 Provider/Service
- 상태 변경은 불변 객체 복사 기반 (`copyWith`)으로 처리
- UI 레이어는 `widgets/`, 게임 로직은 `core/` 등 책임 기반 디렉터리 구성

## 기능 요구사항
### 필수 기능
1. 테트로미노 생성: 7개 블록 무작위 공급, 홀드/다음 블록 미리보기
2. 이동/회전: 좌·우 이동, 소프트 드롭, 하드 드롭, 시계/반시계 회전
3. 충돌 및 고정: 보드 경계, 스택과의 충돌 감지 후 블록 고정
4. 라인 클리어: 한 번에 최대 4줄 제거, 점수 보상 가중치
5. 레벨·속도: 줄 수 기반 레벨 상승, 드롭 속도 가속
6. 점수/통계: 누적 점수, 최고 기록, 플레이 타임, APM 등 기록
7. 입력: 터치 제스처 + 키보드(웹/데스크톱) 지원
8. 사운드/진동: 블록 고정, 라인 삭제, 레벨업 음향 피드백

### 부가 기능
- 튜토리얼/도움말: 기본 조작 안내
- 일시정지/재시작, 재시작 전 확인
- 다크/라이트 테마 전환
- 클라우드 백업(선택): Firebase Sync

## 화면 구조
1. **Splash/Home 화면**
   - 로고 및 시작 버튼, 최고점/최근기록 표시
2. **게임 플레이 화면**
   - 메인 보드, 다음/홀드 블록 패널, 점수/레벨 HUD, 컨트롤 버튼 영역
3. **일시정지 오버레이**
   - 재개, 재시작, 설정, 홈 복귀
4. **설정/튜토리얼 화면**
   - 조작 방식, 사운드 볼륨, 테마, 계정/백업 관리
5. **게임 오버 다이얼로그**
   - 결과 통계, 공유, 다시 시작

## 상태 관리 전략
- Riverpod Provider 계층
  - `GameEngineProvider`: 틱 기반 게임 루프, 블록 생성/충돌 로직
  - `BoardStateProvider`: 현재 보드 그리드, 고정 블록 상태
  - `ScoreProvider`: 점수/레벨/라인 수, 퍼시스턴스와 연동
  - `SettingsProvider`: 입력 스킴, 테마, 사운드 설정
- 상태 전파는 `StateNotifier`/`AsyncNotifier`를 사용해 명시적으로 제어
- 렌더링은 `ConsumerWidget` 또는 `HookConsumerWidget`으로 최소 영역 리빌드
- 애니메이션/타이밍 이벤트는 `TickerProvider`가 아닌 독립 서비스(`GameLoopService`)에서 관리해 테스트 용이성 확보
- 저장소 계층(`repositories/`)을 두어 로컬 스토리지(Firebase/SharedPreferences)와 상태를 분리

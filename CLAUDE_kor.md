# CLAUDE.md

이 파일은 Claude Code(claude.ai/code)가 이 저장소의 코드를 다룰 때 참고할 안내를 제공합니다.

## 프로젝트 개요

ePTM v2(Enhanced Particle Tracking Model)는 새크라멘토–샌와킨 델타를 통과하는 연어 치어의 회유를 시뮬레이션합니다. 캘리포니아 수자원국(DWR)의 DSM2 입자 추적 모델(PTM, Java 패키지 `DWR.DMS.PTM`)을 포크한 것입니다. 이 포크는 어류 행동(유영, 분기점에서의 경로 선택, 포식에 의한 사망)과 개선된 횡방향·연직 방향 랜덤 워크 혼합 기능을 추가합니다. 수리동역학 정보(유속, 수심)는 DSM2 HYDRO 출력(HDF5 형식의 tidefile)에서 가져옵니다. ePTM 자체는 흐름을 계산하지 않습니다. 전체 사용자 가이드는 GitHub Wiki(https://github.com/cvclcm/ePTM_v2/wiki)와 `documentation/RunningEPTMv2.docx`에 있습니다.

## 플랫폼 제약

- **Windows 전용, 32비트 Java 8(JRE 1.8).** `Globals.java`가 `System.loadLibrary("PTM")`를 호출하고, `PTMHydroInput`/`Globals`에 `native` 메서드(tidefile 읽기, 내부↔외부 채널/노드 번호 매핑, 모델 날짜/시간)가 선언되어 있습니다. 이들은 DSM2의 네이티브 `PTM.dll`에 구현되어 있으므로, Linux나 macOS, 또는 64비트 JVM에서는 모델을 실행할 수 없습니다. 이 머신(WSL)에서는 코드 편집과 `javac` 컴파일 정도는 가능할 수 있지만, 시뮬레이션은 실행할 수 없습니다.
- 빌드 시스템(Maven, Gradle, Ant)이 없습니다. 코드는 Eclipse 프로젝트로 빌드합니다(`documentation/ePTM_Eclipse_setup_02mar20.docx` 및 `documentation/videos/7–9*.mp4` 참고). `ptm_SWFSC/lib/`의 jar(jhdf5, opencsv, COM/edu/xml/pj 지원 jar)를 참조하여 컴파일하고, 저장소 루트의 실행 가능한 `eptm.jar`로 내보냅니다. 단순 컴파일 확인은 대략 다음과 같습니다:
  `javac -cp "ptm_SWFSC/lib/*" -d build ptm_SWFSC/DWR/DMS/PTM/*.java`
- 테스트나 린터는 없습니다.

## 실행

진입점은 `DWR.DMS.PTM.MainPTM`입니다. 13개의 위치 인자를 받습니다(`MainPTM.main` 참고):

```
java -Xss5M -Xms512M -Xmx1024M -jar eptm.jar <ePTM.inp> <behavior.h5> <out.h5> <riverBendsDir> <junctionsDir> \
     ChansOrientsBends812.csv Junctions812.csv latVel.csv latEps.csv latdEp.csv latd2Ep.csv VerEps.csv VerdEp.csv
```

두 디렉터리 인자는 보통 둘 다 `configFiles/`를 가리킵니다. `scripts/`의 템플릿 실행 스크립트(모든 경로는 사용자가 채워야 하는 `PATH_TO_*` 자리표시자):
- `runEPTMBasic.py`는 시뮬레이션 1회를 실행합니다. `ePTMRun.py`는 방류/실행 ID 인자를 받아 같은 작업을 합니다. `ePTMBatch.py`는 새 Windows `cmd` 창에서 실행을 띄웁니다. 각 실행은 독립적이므로(어류 간 상호작용 없음) 배치 실행은 완전 병렬(embarrassingly parallel)로 처리할 수 있습니다.
- `runAnimEPTM.py`는 `animateEPTM.jar`(소스: `animateEPTM/plotPTM/readEPTMbinary.java`)로 애니메이션 바이너리를 변환합니다.
- `h5rpkBatch.bat`는 DSM2 hydro tidefile에 `h5repack -l CONTI`를 실행합니다. ePTM이 tidefile을 읽으려면 이 작업을 먼저 해야 합니다.

## 입력 파이프라인

`scripts/ePTMCreateInputs.R`(rhdf5, lubridate, tidyverse)이 실행별 입력 두 가지를 생성합니다:
1. **행동 HDF5 파일.** `channelPars`(`configFiles/ePTMv2CalibValidParams812.csv`의 보정된 채널별 매개변수), `checkpoints`, `swimCode`(11 = 최신 행동 모듈), `nodeDecision`(경로 선택 모델 0–4), `immortal`, 일출/일몰 시각 등을 담고 있습니다. `BehavedParticle`이 `HDF5Factory`로 이름을 기준으로 읽으므로, R 스크립트와 Java 코드의 데이터셋 이름이 일치해야 합니다.
2. **ePTM `.inp` 설정 파일.** DSM2-PTM 텍스트 형식입니다(`configFiles/ePTMConfigTemplate.inp` 참고). DSM2 설정 파일(`${HYDROTIDEFILE}`과 `${TEMPDIR}`을 제공하는 `configFiles/DSM2ConfigTemplate.inp`)을 가리키며, `SCALAR`, `IO_FILE`, `GROUP`, `PARTICLE_FLUX_OUTPUT`, `PARTICLE_INSERTION` 같은 블록을 포함합니다. R 스크립트는 실행 기간을 첫 방류 7일 전부터 마지막 방류 10일 후까지로 설정합니다.

**노드 번호 체계:** 사용자는 방류 노드와 체크포인트를 DSM2 **외부(external)** 노드 번호로 지정합니다. R 스크립트는 `configFiles/nodesIntExt_v8_1_2.csv`를 사용해 체크포인트를 **내부(internal)** 번호로 변환합니다(`channelsIntExt_v8_1_2.csv`도 채널을 같은 방식으로 매핑). 이 둘의 혼동은 과거에 알려진 오류 원인입니다. 새 DSM2 격자를 사용하려면 이 매핑 파일들과 `ChansOrientsBends812.csv`, `Junctions812.csv`를 다시 생성해야 합니다.

## 코드 구조 (`ptm_SWFSC/DWR/DMS/PTM/`)

- 대부분의 파일은 원본 DWR PTM 코드입니다: `PTMEnv`/`PTMFixedInput`/`*FixedData`가 `.inp`를 파싱하고, `PTMHydroInput`이 JNI를 통해 tidefile을 읽으며, `Waterbody` → `Channel`/`Reservoir`/`Conveyor`/`Boundary`와 `Node`, `XSection`이 수로망을 구성합니다. `*Output`/`*Flux*`/`Group*` 클래스가 출력을 처리합니다.
- ePTM 추가 사항(주석에 `VKS` 또는 "Sridharan" 표시가 자주 붙어 있음):
  - `BehavedParticle extends Particle`: 모든 어류 행동을 담당합니다. 행동 HDF5에서 읽은 정적 행동 매개변수, `swimCode`로 분기되는 유영 모드, `nodeDecision`으로 분기되는 분기점 경로 선택, 체크포인트 도착 기록(`checkCheckpoints`), HDF5 출력 쓰기(`IHDF5SimpleWriter`)를 포함합니다.
  - `Particle`: 핵심 이동 및 랜덤 워크 혼합. `MainPTM`이 `Particle.mixCase = 4`(새 혼합 알고리즘)로 설정합니다.
  - `SmartChannel`: 방향 및 만곡 데이터를 가진 채널. `RiverBendsInput`과 `LatData`/`VertData`/`ProfileData`가 `configFiles/lat*.csv` 및 `Ver*.csv` 파일에서 횡방향·연직 방향 유속 분포와 와점성(eddy diffusivity) 분포를 읽어 들입니다.
  - `JunctionsInput`, `DecisionTree`: 분기점별 경로 선택 데이터.
- `MainPTM` 흐름: 전역 변수 및 환경 초기화 → 만곡부와 분기점 데이터 읽기 → `SmartChannel` 방향 갱신 → `BehavedParticle[]` 생성 → 투입 정보 설정 → 시간 단계 루프 → 행동 출력 HDF5 쓰기.

## 저장소 구성

`Data/`에는 보정 및 검증에 사용된 음향 원격측정 데이터셋이 있습니다(`Data/readme` 참고). `documentation/`에는 논문과 튜토리얼 동영상이 있습니다. `modelFiles/`에는 DSM2 격자 KML 파일(내부 및 외부 번호 체계)이 있습니다. `SupportingMaterial/`에는 README 이미지와 보정/검증 발표 자료가 있습니다. 대용량 바이너리(`eptm.jar`, `animateEPTM.jar`, `lib/` 아래의 jar, mp4, pdf)가 git에 커밋되어 있습니다.

# ePTM_v2
향상된 입자 추적 모델 버전 2.0(Enhanced Particle Tracking Model Version 2.0, ePTM v2)은 강에서 산란하고 바다에서 성장하는 어류가 바다 쪽으로 이동하는 과정을 시뮬레이션하는 전산 모델입니다. 이 모델은 과학자, 실무자, 자연자원 관리자가 어류와 같은 수생 생물이 환경을 어떻게 이용하는지 이해할 수 있도록 돕는 가상(synthetic) 어류 원격측정 시스템입니다.

![Alt Text](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/animOutputCropped_1.gif)

모델 설치, 사용법 및 구조에 관한 정보는 모델 Wiki 페이지에서 확인할 수 있습니다: [Link](https://github.com/cvclcm/ePTM_v2/wiki).

모델에 관해 질문이 있으시면 *vamsi.sridharan@noaa.gov* 로 문의해 주십시오.


# 목차
1. 소개
2. 활용 사례
3. 배경
4. 컴퓨팅 요구 사항
6. 프로젝트 정보

# 소개
이 모델은 구체성의 정도에 따라 다양한 방식으로 설명할 수 있습니다.

이 모델은 **의사결정 지원 도구**입니다. 기후 변화, 해수면 상승, 노후화된 기반시설, 토지 이용 및 토지 피복 변화 등으로 인한 불확실성 하에서, 다목적 지표수 시스템의 물 공급에 관한 관리 및 정책 결정을 내리는 데 필요한 과학적 통찰을 제공하는 데 현재 사용되고 있으며, 또 사용될 수 있기 때문입니다.

이 모델은 **에이전트 기반 모델**의 일종으로, 시뮬레이션된 어류가 일련의 행동 규칙, 즉 "에이전트"에 따라 주변 수생 환경과 상호작용합니다.

또한 넓은 공간 영역에 걸쳐 개별 가상 어류의 회유 생활사를 추적한다는 점에서 **개체 기반 모델**이기도 합니다. 다만 현재 형태의 모델은 어군 내부의 상호작용이나 포식자와 피식자 간의 상호작용을 명시적으로 시뮬레이션하지는 않는다는 점에 유의해야 합니다.

이 모델은 실제 어류 관측에 근거한 물리적·생물학적 제약 조건을 사용하여 가상 어류의 이동을 결정하는 **과정 기반(메커니즘) 모델**입니다. 구체적으로는 국지적 환경(여기서는 물의 흐름과 하루 중 시간)의 영향을 받는 가상 어류의 자기상관적 이동을 시뮬레이션하는 입자 추적 모델, 즉 확률적 랜덤 워크 모델입니다.

마지막으로, 이 모델은 **완전한 데이터 기반 모델**입니다. 복잡한 환경을 통한 동물의 회유는 본질적으로 다중 스케일 과정이며, 본 모델의 이동 규칙을 정의할 때 이동하는 어류에 대해 확보 가능한 최선의 음향 원격측정 데이터를 채택하였습니다.

**새크라멘토 강의 수로 분기점에서 고해상도 음향 태그를 부착한 치누크 연어의 이동 궤적과 태그된 포식자에 대한 노출 자료를 통해 중규모(mesoscale) 행동을 연구하고 가상 어류의 행동 규칙을 정의할 수 있습니다**

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/Tracks2011.jpg) 

**새크라멘토 강에서 바다 쪽으로 이동하는 음향 태그 치누크 연어의 회유 통계는 모델의 행동 매개변수 보정에 사용되는 거시적 패턴을 보여줍니다** 

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/MigrationRateVsReach.jpg)

이 모델은 연어과 어류의 이동에 관한 과학 문헌의 철저한 검토, 이해관계자 참여, 그리고 독창적인 과학 연구를 통해 개발되었습니다.
어류의 중규모 이동 행동에 관한 근본적인 발견은 최근 Olivetti et al. (2021) 논문에서 보고하였습니다: [Link](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.13604?casa_token=R24dUwXx0FcAAAAA%3AUYvokc4iyCMO8VpBF6dAU1C306BvSg4r60KTjsV27wgm0W_EKtuH34H8F6m2rLX9-JD7LwvseX23zHU).
하구를 통한 연어과 어류 이동의 거시적 표현은 최근 Sridharan et al. (2019) 논문에서 개발 및 보고하였습니다: [Link](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1029/2019WR025429?casa_token=8r5Cth5KgH4AAAAA%3Ac67189ZeR9wPmDHx6iglofh9-e719x0znYf-Svraj5li_6-EaScwiWG_0wH9kZhUKxO3HX8V31_CEfo).

## ePTM v2가 왜 필요한가?
회유성 수생 동물이 복잡한 지표수 환경을 어떻게 이동하는지 이해하고자 하는 과학자이거나, 새로운 수자원 사업이나 정책이 지역 야생생물에 어떤 영향을 미칠지 알아야 하는 수자원 관리자 또는 보전 정책 결정자라면, 현재 일어나고 있는 일을 이해하는 데 도움이 되는 민첩하고 신뢰할 수 있는 모델이 필요합니다. 음향 원격측정 프로그램은 운영 비용이 많이 들고 단편적인 데이터만 제공합니다. 데이터 범위가 제한된 지역에서는 어떤 일이 일어날까요? 유량과 같은 거시적 시스템 변수와 생존율을 연결하는 통계 모델이면 충분하다고 생각할 수도 있습니다. 하지만 기후 변화 및 기타 요인으로 인한 불확실성 때문에 통계적 관계가 변한다면 어떻게 될까요? ePTM과 같은 메커니즘 모델은 새로운 환경 조건에서 어류에게 일어날 가능성이 높은 일을 예측할 수 있습니다. 아래는 모델 검증 결과(보정 과정에서 제외해 둔 데이터와 모델 성능의 비교)입니다.

**새크라멘토-샌와킨 델타의 겨울 회귀(Winter-run) 치누크 연어 치어에 대한 유량-생존율 관계의 표본 외(out of sample) 검증. 낮은 편향(<0.1), 음향 원격측정 데이터와 모델 결과의 95% 신뢰구간 간의 상당한 중첩, 그리고 모델이 기능적으로 유사한 관계를 재현한다는 점은 모델의 가치를 보여줍니다**

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/FlowvsSurvival.jpg)

**창조(밀물) 대 낙조(썰물) 시 여러 회유 경로로 유입될 확률에 대한 표본 외 검증은 모델이 회유 경로를 잘 예측할 수 있음을 보여줍니다. 2014년에 조지아나 슬라우(Georgiana Slough) 입구에 부유식 어류 유도 구조물이 설치되었으며, 모델에서는 이를 근사적으로만 표현하였습니다**

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/RoutingProbabilties.jpg)

# 활용 사례
현재 ePTM v2는 캘리포니아 중부의 새크라멘토-샌와킨 델타를 거쳐 샌프란시스코 만과 태평양으로 이동하는 치누크 연어 치어에 대해 보정되어 있습니다. 이 모델은 하천 흐름과 조류에 대한 수치 시뮬레이션을 필요로 하며, 이는 캘리포니아 수자원국(DWR)의 델타 시뮬레이션 모델(DSM2) v.8.1.2가 제공합니다. DSM2는 DWR이 다음 주소에서 호스팅하고 있습니다: [Link](https://github.com/CADWRDeltaModeling/dsm2).

**새크라멘토 강 겨울 회귀 치누크 연어 스몰트(smolt) 회유 적용을 위한 ePTM v2 모델 영역(새크라멘토 강 강조 표시)**

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/DeltaMapwSacRiver.jpg)

캘리포니아 센트럴 밸리에서 이 모델은 새크라멘토-샌와킨 델타에서 성장하고 회유하는 멸종위기종 새크라멘토 강 겨울 회귀 치누크 연어 치어의 월별 생존율 추정치를 제공하는 데 사용됩니다. 이 추정치는 더 큰 단계 구조(stage-structured) 개체군 동태 모델인 센트럴 밸리 생활사 모델(Central Valley Life Cycle Model)에서 사용됩니다: [Link](https://oceanview.pfeg.noaa.gov/wrlcm/intro).

현재 이 모델은 다른 수리동역학 엔진과 함께 실행되도록 구성되어 있지 않습니다. 따라서 여러분의 수계에 ePTM v2와 DSM2를 사용한 모델을 개발하려면, 모델 격자를 생성하고, 이 격자를 이용해 유량 예측용 DSM2 모델을 보정 및 검증한 후, ePTM v2에서 사용할 수 있는 수리동역학 출력을 생성해야 합니다.

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/SacRiverWinterRunChinookSmolt_SiskoJakeUSFWS.jpg)

**새크라멘토 강 겨울 회귀 치누크 연어 스몰트 (사진 제공: Jake Sisko, 미국 어류야생동물관리국)**

마찬가지로, 현재 이 모델은 연어과 치어의 회유를 시뮬레이션하기에 적합한 행동 규칙을 갖추고 있으며, Perry (2010; 2012)가 발표한 데이터를 사용하여 캘리포니아 새크라멘토 및 샌와킨 수계의 치누크 연어에 대해 보정 및 검증되었습니다 [Link](https://github.com/cvclcm/ePTM_v2/tree/main/documentation). 모델 보정 및 검증에 대한 자세한 설명은 여기에서 확인할 수 있습니다: [Link](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/Sridharan_ePTMv2_CalibrationValidation_Dec1_2021.pptx). 모델 구축, 보정 및 검증에 사용된 모든 데이터는 다음에 보관되어 있습니다: [Link](https://github.com/cvclcm/ePTM_v2/tree/main/Data).

소스 코드를 수정하여 행동 규칙을 변경하고, 다른 수계의 다른 어종에 대해 모델을 재보정할 수 있습니다. 방법은 Wiki 페이지를 참고하십시오: [Link](https://github.com/cvclcm/ePTM_v2/wiki).

# 배경
ePTM v2는 중성 부력의 수동적 추적자를 시뮬레이션하는 DWR의 입자 추적 모델(Particle Tracking Model, PTM)에 행동 클래스를 추가한 것입니다. PTM과 ePTM v2는 모두 유속과 수심을 시뮬레이션하기 위한 수리동역학 엔진이 필요하며, 이는 1차원 천수 방정식 해석기인 DSM2 모델이 담당합니다. 모델에 대한 상세한 동료 검토는 Sridharan et al. (2018) 논문에서 확인할 수 있습니다 [Link](https://escholarship.org/uc/item/0vm955tw). ePTM v2는 또한 PTM의 기반 랜덤 워크 알고리즘에 대한 상당한 수치적 개선을 포함하고 있으며, 그 대부분은 Sridharan et al. (2017) 논문에 자세히 설명되어 있습니다 [Link](https://ascelibrary.org/doi/full/10.1061/(ASCE)HY.1943-7900.0001399?casa_token=yMf5O160xyoAAAAA:v891cN9CzxrTTSQBoi3FTBvZsfMKstbIU1Et8QPf5Dh6dHIJsE-wh8eotqCa2S-8X-MV4hgnXA).

ePTM v2는 수생 동물 이동의 세 가지 측면을 시뮬레이션합니다: 유영에 의한 회유, 조석과 물 관리 운영으로 인한 흐름 역전이 존재하는 하천 분기점 및 수로 합류점과 같은 복잡한 지형에서의 경로 선택, 그리고 포식에 의한 사망입니다. 이러한 메커니즘의 세부 사항은 여기서 다루기에는 너무 복잡하지만, 모델 구조에 대한 간략한 설명은 다음에서 확인할 수 있습니다: [Link](https://oceanview.pfeg.noaa.gov/wrlcm/documents/documentation/ePTM_Study_Plan_FINAL_02_12_2021_v2.pdf). 이 문서에는 모델이 다양한 수자원 관리 분야에서 어떻게 계속 사용되고 있고 사용될 수 있는지에 대한 정보도 담겨 있습니다. 모델의 생물학적 행동 규칙을 설명하는 동영상도 제작하였으며, 여기에서 볼 수 있습니다: [Link](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/Sridharan_04_09_21_BDSC_ePTMV2Description.mp4).

## 모델 개발 과정
이 과학 모델은 다음과 같은 핵심 원칙에 따라 개발되었습니다:
1. 여러 운동 스케일에 걸쳐 자기 일관성을 유지하고, 현실적인 물리적·생물학적 과정을 포함할 것
2. 수십 년 규모의 개체군 동태 시뮬레이션에 활용할 수 있을 만큼 민첩하고 확장 가능해야 하며, 최첨단의 수치적 정교함과 현실성을 유지하면서도 단독으로 또는 더 큰 모델링 생태계의 구성 요소로 사용할 수 있을 만큼 다재다능할 것
3. 고품질 과학 데이터, 다학제 개발팀, 그리고 캘리포니아 센트럴 밸리의 분야 전문가 및 의사결정자와의 적극적인 이해관계자 참여를 활용하는 다기관 협력을 통한 포용적이고 투명한 과정에서 탄생할 것

ePTM v1.0은 미국 국립해양수산청(NMFS), 캘리포니아 어류야생동물국(CDFW), 미국 지질조사국(USGS)의 협력으로 개발되었습니다. 이 모델은 DSM2-PTM 모델을 확장하여 다양한 어류 행동을 포함시켰습니다. 이후 모델을 치누크 연어 생활사 모델에 통합하고 모델 구상을 위한 이해관계자 참여를 지원해 준 미국 개척국(Bureau of Reclamation)과, 다중 스케일 데이터 소스를 결합하여 연어과 어류의 회유 이동에 대한 보다 근본적인 이해를 발전시키도록 지원해 준 캘리포니아 어류야생동물국의 아낌없는 후원으로, 개선된 흐름의 수리동역학적 표현과 보다 현실적인 행동 모델을 갖춘 버전 2.0이 개발되었습니다.

모델 개발 과정 전반에 걸쳐 지역, 광역 및 연방 이해관계자 커뮤니티와 소통해 왔습니다. 소통 이력과 이해관계자가 제안한 행동 가설(모델로 다양한 평가 단계에 있음)은 다음 주소의 *Workshops* 및 *Stakeholder shared documents* 탭에서 각각 확인할 수 있습니다: [Link](https://oceanview.pfeg.noaa.gov/wrlcm/resources).

# 컴퓨팅 요구 사항
ePTM v2는 Java로 개발되었으며 현재 Windows PC에서만 실행됩니다. 단일 실행 또는 배치 모드로 실행할 수 있습니다. 모델 실행에는 다음과 같은 하드웨어 및 소프트웨어가 필요합니다:

1. 수개월 또는 수년간의 시뮬레이션을 실행할 수 있는 고성능 노트북 또는 데스크톱, 그리고 수십 년 규모의 시뮬레이션을 위한 고성능 컴퓨터, 클러스터 또는 클라우드 컴퓨팅
2. 모델 컴파일 및 실행을 위한 32비트 Java 런타임 환경 8 [![Image](https://img.shields.io/badge/java-jre%201.8-blue)](https://www.java.com/download/ie_manual.jsp)
3. 모델 소스 코드 작업을 위한 Eclipse와 같은 IDE 환경. 아래 설치 가이드에서 Eclipse 사용법을 안내합니다: [Link](https://www.eclipse.org/downloads/packages/release/juno/sr1/eclipse-ide-java-ee-developers). **32비트 Java 8을 지원하는 IDE가 필요하다는 점에 유의하십시오.** 다음에서 찾을 수 있습니다 [![Image](https://img.shields.io/badge/Eclipse-4.16-red)](https://archive.eclipse.org/eclipse/downloads/).
4. 모델의 수리동역학 엔진인 DSM2 [![Image](https://img.shields.io/badge/DSM2-v.8.1.2-orange)](https://water.ca.gov/Library/Modeling-and-Analysis/Bay-Delta-Region-models-and-tools/Delta-Simulation-Model-II) 
5. 수리동역학 모델 결과의 시각화 및 처리를 위한 HDF5 도구 모음: [Link](https://www.hdfgroup.org/solutions/hdf5/)
6. Notepad++와 같은 좋은 텍스트 편집기: [Link](https://notepad-plus-plus.org/downloads/)
7. 모델 실행 및 출력 처리 스크립트 작성을 위한 Python 3.x [![Image](https://img.shields.io/badge/python-3.7%2B-green)](https://www.python.org/downloads/) 
8. 모델 결과 분석을 위한 R 4.x 또는 Matlab 2018b 이상 [![Image](https://img.shields.io/badge/R-4.0%2B-yellow)](https://www.r-project.org/)
9. DSM2의 수리동역학 시계열을 시각화하기 위한 HEC-DssVue: [Link](https://www.hec.usace.army.mil/software/hec-dssvue/)
10. DSM2 격자 시각화를 위한 Google Earth Pro (캘리포니아 센트럴 밸리에서 사용자 맞춤형으로 모델을 설정할 때 매우 유용함): [Link](https://www.google.com/earth/download/gep/agree.html?hl=en-GB)

# 프로젝트 정보
여기에서 프로젝트의 최신 소식, 정보 및 현황을 업데이트합니다.

## 소식

![Image](https://img.shields.io/static/v1?label=01/12&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
ePTM 모델 개발을 일부 지원한 Proposition 1 프로젝트에 대해 캘리포니아 어류야생동물국에 제출한 최종 보고서를 이제 여기에서 볼 수 있습니다: [Link](https://github.com/cvclcm/ePTM_v2/blob/main/documentation/Raimondi_et_al_2021_CDFW_FinalReport.pdf). 다음과 같이 인용해 주십시오:

Raimondi, P., Hein, A., Sridharan, V., Danner, E., and Lindley, S. 2021. A next-generation model of juvenile salmon migration through the Sacramento-San Joaquin Delta. Report submitted by the University of California Santa Cruz to the California Department of Fish and Wildlife Under Proposition 1 Scientific Award No. P1896007. Sacta Cruz, CA. 55p. Available at: https://github.com/cvclcm/ePTM_v2/blob/main/documentation/Raimondi_et_al_2021_CDFW_FinalReport.pdf.  

![Image](https://img.shields.io/static/v1?label=12/01&message=2021&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
ePTM v2는 현재 캘리포니아 델타 수로 사업(Delta Conveyance Project)에서 기후 변화 및 해수면 상승 조건 하의 대안적 물 관리 운영 평가에 사용되고 있습니다.

![Image](https://img.shields.io/static/v1?label=01/06&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
ePTM v2가 Maven's Notebook 기사에 소개되었습니다:

https://mavensnotebook.com/2022/01/06/bay-delta-science-conference-applying-the-winter-run-life-cycle-model-to-pressing-hydromanagement-questions-in-the-central-valley/

![Image](https://img.shields.io/static/v1?label=06/28&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
*scripts* 폴더의 ePTMCreatInputs.R과 Wiki의 사용법에 여러 수정 사항이 있으니 참고하십시오. 체크포인트를 지정할 때는 DSM2 *외부(External)* 노드 번호를 사용하십시오. 이전에는 *내부(Internal)* 노드 번호로 잘못 안내되었습니다. 또한 ePTM 실행을 위한 기본 스크립트와 Wiki의 설명이 간소화되었습니다.

![Image](https://img.shields.io/static/v1?label=06/28&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
ePTM v2는 캘리포니아 SITES 저수지 사업에서 기후 변화 및 해수면 상승 조건 하의 대안적 물 관리 운영 평가에 사용될 예정입니다.

![Image](https://img.shields.io/static/v1?label=07/05&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
2022년 6월 30일 이해관계자 참여 워크숍의 모든 교육 동영상을 이제 documentation/videos 섹션에서 볼 수 있습니다.

![Image](https://img.shields.io/static/v1?label=07/13&message=2022&labelColor=lightgrey&color=brightgreen&style=plastic&logo=githubactions)
이 readme 파일의 업데이트 사항을 참고하십시오. 이전에 ePTM 실행에 DSM2 v8.1.2가 필요하다고 잘못 안내하였습니다. 필요한 전제 조건만 갖추어지면 ePTM은 어떤 DSM2 버전에서도 실행할 수 있습니다.

## 향후 개선 사항
모델을 개선할 수 있는 몇 가지 영역이 있습니다:

1. 어류는 흐름에 따라 단순히 떠다니는 수동적 입자처럼 행동하지 않는다는 것이 알려져 있으므로(Sridharan et al. 2017; Hance et al. 2020; Gross et al. 2021), 가상 어류가 난류 에디에 의해 단순히 밀려나도록 두는 대신, 국지적 흐름에 조건부인 관측 분포에서 횡방향 및 연직 방향 랜덤 워크 이동을 추출하도록 갱신해야 합니다.
2. 복잡한 지표수 시스템에서 이동이 주로 2차원 및 3차원적인 곳과, 1차원 하천 방향 이동 시뮬레이션으로 충분한 곳을 연구함으로써 추가적인 효율성을 얻을 수 있습니다.
3. 실험실 수로(flume)에서 얻은 실험적 흐름 분포 대신, 다양한 흐름 조건에서 모델의 실제 물리적 영역 내에서 측정된 유속을 사용합니다.
4. 64비트 Java를 사용하도록 코드를 갱신합니다.
5. Linux와 Mac에서도 실행되도록 코드를 갱신합니다.
6. 기반 수리동역학 엔진과 독립적으로 실행될 수 있도록 코드가 DSM2와 상호작용하는 방식을 갱신합니다.

이러한 개선에 기여하고 싶으신가요? 그렇다면 코드를 여러분의 저장소로 포크(fork)하고, 안정적인 빌드가 준비되면 풀 리퀘스트(pull request)를 보내 주십시오.

## 핵심 팀
핵심 모델 개발팀은 다음 기관으로 구성됩니다:

캘리포니아 대학교 산타크루즈: 해양과학연구소(Institute of Marine Sciences)

미국 해양대기청(NOAA) 국립해양수산청(NMFS) 남서수산과학센터(Southwest Fisheries Science Center)

미국 지질조사국(USGS) 서부수산연구센터(Western Fisheries Research Center)

QEDA Consulting, LLC.

## 감사의 글
핵심 모델링 팀은 프로젝트에 크게 기여한 캘리포니아 수자원국(DWR)에 감사드립니다.

또한 프로젝트에 기술 및 프로그램 지원을 제공한 캘리포니아 대학교 산타크루즈의 수산 협력 프로그램(Fisheries Cooperative Program)에도 감사드립니다.

모델 구축, 보정 및 검증에 사용된 데이터를 제공해 준 미국 지질조사국, 캘리포니아 어류야생동물국, 미국 어류야생동물관리국, 그리고 시애틀 워싱턴 대학교에 감사드립니다.

또한 초기 모델 개발 과정에 반영한 지역, 광역 및 연방 기관과 학계 및 산업계 이해관계자들의 분야 전문 지식에도 감사의 뜻을 전합니다.

## 연구비 지원
ePTM v2 개발의 과학적 기초에 대한 주요 지원은 캘리포니아 어류야생동물국이 수여한 캘리포니아 Proposition 1 과학 연구 보조금으로 제공되었습니다.

캘리포니아 센트럴 밸리의 의사결정 지원을 위한 모델 통합 및 이해관계자 참여에 대한 주요 지원은 캘리포니아 대학교 산타크루즈와의 협력 협정을 통해 미국 개척국(Bureau of Reclamation)이 제공하였습니다.

**연구비 지원 기관**

![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/CDFWLogo.png) ![Image](https://github.com/cvclcm/ePTM_v2/blob/main/SupportingMaterial/BORLogo.png)

# DataLake & 데이터 분석 워크샵

## 1. Introduction 
<!-- 15분 -->

### 1-1. 시작하기 전

이 핸즈온랩을 통해 여러분들은 1) 데이터 분석을 위한 데이터 클랜징과 정규화 2) 쿼리를 통한 데이터 분석과 시각화 작업 3) 데이터 예측 과정을 살펴볼 예정입니다. 

Scenario : ABC 회사는 온라인 반려동물 제품 소매업체입니다. 최근 반려동물을 위한 새로운 제품 라인을 출시했습니다. 큰 성공을 거둘 것으로 예상했지만 일부 지역의 매출이 기대에 미치지 못했습니다. 마케팅 조사에 따르면 해당 지역에서 제품에 대한 인지도가 낮은 것이 판매 부진의 주요 원인인 것으로 나타났습니다. 이 문제를 해결하기 위해 해당 지역에서 타겟팅된 메일 캠페인을 시작하여 인지도를 높이면서 제품 판매를 늘리려고 합니다. 팀은 코딩을 하거나 인프라를 관리할 시간이 많지 않기 때문에 손쉽게 해결할 수 있는 방법이 필요합니다.

Action : 팀은 1) 고객/제품/판매 데이터를 분석 및 변환하여 두 제품 라인의 판매를 비교하고, 2) 특정 제품을 마케팅할 고객의 우편 번호를 파악해야 합니다.

데이터 변환 단계:
1. 고객 데이터 (Customer) : 고객 데이터를 클랜징하고 주소 열에서 우편 번호를 분석합니다.
1. 판매 데이터 (Sales): 고객 데이터 및 제품 데이터와 Join하여 우편번호 및 제품 유형별로 판매량을 비교합니다.
1. 제품 데이터 (Product) : 마지막으로 판매 가능한 제품 ID가 포함된 우편번호 목록을 생성합니다.

 **Data Model**
    - ![](images/datamodel.png)

 **Logical data flow**
    - ![](images/dataflow.png)


### 1.2 이벤트 계정으로 AWS 콘솔 접속 하기

1. AWS Wokshop Portal에 로그인하여 실습을 진행하실 경우 Team Hash 값이 필요합니다. 여기를 클릭 한 후, 이벤트 주최자로부터 받은 12자리 Participant Hash 값을 입력하면 오른쪽 하단 버튼이 Accept Terms & Login으로 변경됩니다. 다음 단계로 넘어가기 위해 해당 버튼을 클릭합니다.
    - ![](images/setting_up-img1.png)

1. Email One-Time Password (OTP) 버튼을 클릭합니다.
    - ![](images/1EventEngineSignInOptions.png)

1. 본인의 이메일 계정을 입력하고 Send Code 버튼을 클릭합니다.
    - ![](images/2EventEngineSpecifyEmail.png)

1. 작성한 이메일 수신함에서 제목이 Your one-time passcode 인 이메일을 확인하고 passcode를 복사합니다. 복사한 passcode를 아래와 같이 붙여넣기 한 뒤, Sign in 버튼을 클릭합니다.
    - ![](images/3EventEngineSpecifyPasscode.png)

1. 다음 화면에서 AWS Console 버튼을 누르면 AWS 관리콘솔에 로그인할 수 있는 로그인 링크를 받을 수 있습니다.
    - ![](images/4EventEngineTeamDashboard.png)

1. Open AWS Console 버튼을 누르면 AWS 관리콘솔로 접속할 수 있습니다. 또한, CLI 환경을 위한 "Access Key" 와 "Secret Access Key" 도 확인할 수 있습니다.
    - ![](images/5EventEngineConsoleLogin.png)

위의 단계를 모두 수행했다면 이제 실습을 시작할 수 있습니다.

### 1.3 S3버킷 생성하기(CloudFormation)
<!--  5분 -->

1. 워크샵을 시작하기 전에 필요한 AWS 리소스를 생성해야 합니다. 이를 위해 리소스가 포함된 스택을 생성할 수 있는 AWS CloudFormation 템플릿을 제공합니다. 스택을 생성하면 AWS가 계정에 여러 리소스를 생성합니다. 이 워크샵에서는 사용할 데이터 파일 및 폴더와 함께 접두사가 glue-databrew-immersionday인 S3 버킷을 생성합니다. 
아래의 Launch Stack 버튼을 클릭하여 이 워크샵에 필요한 리소스를 생성합니다.

    - [![Launch Stack](https://cdn.rawgit.com/buildkite/cloudformation-launch-stack-button-svg/master/launch-stack.svg)](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://aws-data-analytics-workshops.s3.amazonaws.com/glue-databrew-immersionday-v2/databrew_ID-prod.yaml&stackName=glue-databrew-immersionday)
    - ![](images/createstack.png)

CloudFormation 스택을 완료하는 데 대략 3~5분 정도 소요됩니다.
<!-- 15:52:46 15:54:06 금방이네 -->
1. 스택 생성이 성공하면 **Outputs** 탭에서 새로 생성된 버킷 이름을 확인할 수 있습니다.
    - ![](images/cf-complete.png)

1. 상단 검색창에 **S3**로 검색하여, S3 서비스로 이동 후 버킷 메뉴를 클릭하면 glue-databrew-immersionday 버킷을 확인할 수 있습니다.
    - ![](images/s3-bucket-name.png)

1. glue-databrew-immersionday 버킷에는 다음과 같은 구조를 가지고 있습니다.
    - ![](images/s3-bucket.png)

## 2. Profiling and Data Quality

AWS Glue DataBrew는 데이터 패턴을 이해하고 이상 징후를 감지하기 위해 데이터를 프로파일링하여 데이터의 품질을 평가할 수 있도록 도와줍니다. 데이터 세트의 데이터 프로필 개요 섹션에서 데이터에 대한 통계 요약을 검토하고 수집할 수 있습니다.
   - ![](images/profiling.png)

아래 내용들을 확인해보겠습니다.

- Data Profile Job 만들기
	- 개인 식별 정보(PII) 데이터 탐지
	- Data Quality(DQ) 검사
- Map Data Lineage

### 2.1 Customer Dataset

Dataset은 단순히 열 또는 필드로 나뉜 데이터 행 또는 레코드 집합을 의미합니다. DataBrew는 형식이 지정된 파일에서 가져온 모든 소스의 데이터로 작업할 수 있으며, 점점 늘어나는 데이터 저장소 목록에 직접 연결할 수 있습니다. DataBrew에서 데이터 집합은 데이터에 대한 읽기 전용 연결입니다. DataBrew는 데이터를 참조하기 위해 일련의 설명 메타데이터를 수집합니다. 실제 데이터는 DataBrew에서 변경하거나 저장할 수 없습니다. 간단히 설명하기 위해 Dataset은 실제 Dataset과 DataBrew가 사용하는 메타데이터를 모두 의미합니다.

이 실습에서는 Customers dataset을 만듭니다. 아래는 샘플 고객 데이터입니다.
 
1. [AWS Glue DataBrew](https://console.aws.amazon.com/databrew/home?region=us-east-1#)서비로 이동합니다. 오른쪽 상단에 **미국 동부 (버지니아 북부) us-east-1** 리전을 사용하고 있는지 확인합니다.
   - ![](images/checkregion.png)

1. 왼쪽 메뉴에서 **Datasets**를 선택합니다.
   - ![](images/checkregion.png)

1. **Connect new dataset**을 선택합니다.
   - ![](images/create_a_dataset.png)

1. dataset의 이름을 Customers로 지정합니다.

1. 서비스로 Amazon S3를 선택합니다.

1. Enter your source from S3 필드에 s3://glue-databrew-immersionday를 입력합니다. CloudFormation 템플릿으로 생성한 버킷을 선택합니다.

1. datafiles > customers folder 로 이동합니다.

1. "customer.csv" 파일을 선택합니다.
   - ![](images/dataset_details.png)

1. file type으로 CSV를 선택합니다.

1. 쉼표를 CSV 구분 기호로 선택합니다.

1. 첫 번째 행을 헤더로 처리를 선택합니다.

1. 오른쪽 아래에 있는 Create dataset 버튼을 선택합니다.
   - ![](images/dataset_type_as_csv.png)

1. Customers Dataset이 생성됩니다.
   - ![](images/create_a_dataset.png)

1. Customers Dataset을 선택하여 고객 데이터를 미리 확인해봅니다.
   - ![](images/datasetpreview.png)


다음으로 DataBrew 프로젝트를 생성합니다.


### 2.2 Customer Profile Job (PII)

프로파일링 작업은 dataset에 대해 다양한 평가를 실행합니다. 데이터 프로파일링이 수집하는 정보는 어떤 종류의 데이터 준비 단계가 필요한지 결정하는 데 도움이 됩니다.
이 섹션에서는 프로필 작업에서 PII 탐지 기능을 활성화하여 데이터 집합에 있는 민감한 개인 식별 정보(PII) 데이터를 식별합니다.


1. 왼쪽 탐색 창에서 **Datasets**을 선택합니다.
1. 이전 단계에서 만든 **Customers** dataset을 선택합니다.

1. 오른쪽 상단에서 **Run data profile**을 선택합니다.

   - ![](images/create_a_customer_profile_1.png)

1. **Create a profile job**를 선택합니다.
   - ![](images/create_a_customer_profile_2.png)


1. Job 이름으로 *Customers profile job* 입력

2. **Full dataset** 선택
   - ![](images/create_a_customer_profile_3.png)

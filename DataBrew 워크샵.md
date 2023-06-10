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

Data Model
   ![](images/datamodel.png)

Logical data flow
   ![](images/dataflow.png)


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
 
1. [AWS Glue DataBrew](https://console.aws.amazon.com/databrew/home?region=us-east-1#)서비로 이동합니다.
오른쪽 상단에 **US East (N. Virginia) us-east-1** 리전을 사용하고 있는지 확인합니다.
   - ![](images/checkregion.png)

1. 왼쪽 메뉴에서 **Datasets**를 선택합니다.

1. **Connect new dataset**을 선택합니다.
   - ![](images/create_a_dataset.png)

1. dataset의 이름을 *Customers*로 지정합니다.

1. 서비스로 **Amazon S3**를 선택합니다.

1. **Enter your source from S3** 텍스트 박스에 *s3://glue-databrew-immersionday*를 입력합니다. CloudFormation 템플릿으로 생성한 버킷을 선택합니다.

1. **datafiles > customers** 폴더로 이동합니다.

1. "customer.csv" 파일을 체크합니다.
   - ![](images/dataset_details.png)

1. file type으로 **CSV**를 선택합니다.

1. **Comma(쉼표)**를 CSV 구분 기호로 선택합니다.

1. **Treat first row as header(첫 번째 행을 헤더로 처리)**를 선택합니다.

1. 오른쪽 아래에 있는 **Create dataset 버튼**을 선택합니다.
   - ![](images/dataset_type_as_csv.png)

1. Customers Dataset이 생성됩니다.
   - ![](images/datasetcreated.png)

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


1. Job name 텍스트 박스에 *Customers profile job* 입력합니다.

1. **Full dataset** 선택합니다.
   - ![](images/create_a_customer_profile_3.png)


1. Browse 버튼을 클릭하여 **glue-databrew-immersionday-xxxx S3 버킷 > profile-output 폴더**를 선택 후 Select 버튼을 클릭합니다.
   - ![](images/create_a_customer_profile_4.png)

1. **Data profile configurations**을 확장하고, 데이터 프로필 작업을 실행할 때 PII 열을 식별하려면 **PII statistics**에서 **Enable PII statistics**을 선택합니다. 추가로 **PII categories**에서 **All categories**를 선택합니다.
   - ![](images/create_a_customer_profile_5.png)


1. **Permissions** 섹션으로 건너뛰고, **Role name** 드롭다운하여 **Create new IAM role**를 선택합니다.

1. **New IAM Role suffix** 텍스트 박스에 *ID*를 입력합니다. 그러면 AWSGlueDataBrewServiceRole-ID라는 새 IAM role이 생성됩니다.

1. **Create and run job**을 선택합니다.

   - ![](images/create_a_customer_profile_6.png)


1. 그러면 **Customers dataset**의 **Data profile overview** 탭으로 이동합니다.
작업이 완료되는 데 약 5분이 소요됩니다.
   - ![](images/create_a_customer_profile_7.png)


1. 이 보고서는 컬럼 통계와 함께 PII 대상으로 확인된 PII 컬럼의 카탈로그를 제공합니다. 또한 검토할 수 있는 잠재적인 PII 열을 보여줍니다.
이후 과정에서는 변환을 통해 확인된 PII 열을 선택적으로 삭제합니다.

다음으로, Sales Dataset 및 Data Quality Rules을 생성합니다.


### 2.3 Sales Dataset

이 실습에서는 Sales 데이터 집합을 만듭니다. 아래는 샘플 판매 데이터입니다.
   - ![](images/salesdatasample.png)

1. 왼쪽 메뉴에서 **Datasets**을 선택합니다.

1. **Connect new dataset**을 선택합니다.
   - ![](images/connectnewdataset.png)

1. dataset의 이름을 *Sales*로 지정합니다.

1. 서비스로 **Amazon S3**를 선택합니다.

1. **Enter your source from S3** 텍스트 박스에 *s3://glue-databrew-immersionday*를 입력합니다. CloudFormation 템플릿으로 생성한 버킷을 선택합니다.

1. **datafiles > sales** 폴더를 선택합니다.

   - ![](images/Create_a_sales_dataset_1.png)

1. file type으로 **CSV**를 선택합니다.

1. **Comma(쉼표)**를 CSV 구분 기호로 선택합니다.

1. **Treat first row as header(첫 번째 행을 헤더로 처리)**를 선택합니다.

1. 오른쪽 아래에 있는 **Create dataset 버튼**을 선택합니다.
   - ![](images/Create_a_sales_dataset_2.png)

1. Sales Dataset이 생성됩니다. Sales Dataset을 선택하여 고객 데이터를 미리 확인해봅니다.
   - ![](images/salesdataset.png)

다음으로 data quality rules과 data profiling job을 만들 것입니다.

### 2.4 Sales Profile Job (DQ)

이 실습에서는 Sales dataset의 data quality을 확인하고, data quality ruleset을 만든 다음 profile job을 실행하여 적용합니다.

1. 왼쪽 메뉴에서 **DQ Rules**을 선택하고, **Create data quality ruleset**를 클릭합니다.
    - ![](images/create_sales_dq_ruleset_1.png)

1. **ruleset name** 텍스트 박스에 *Sales DQ Checks*로 지정합니다.

1. **Associated datase** 섹션에서 **Sales dataset**을 선택합니다. **View associated dataset details**를 클릭하여 dataset을 미리 확인합니다.
1. 이제 dataset을 미리 볼 수 있으며, **Sales dataset의 (Quality, Total_Sales)컬럼**에 data quality 문제가 있음을 확인할 수 있습니다.
    - ![](images/create_sales_dq_ruleset_2.png)

1. 또한 적용할 수 있는 data quality check에 대한 **Recommendations(권장 사항)**도 확인할 수 있습니다.
    - ![](images/create_sales_dq_ruleset_3.png)

- DATASET QUALITY CHECKS
  중복 행이 있는 dataset 확인

- COLUMN QUALITY CHECKS
  모든 컬에 missing values 0%인지 확인

여러 규칙을 추가할 수 있으며, 각 규칙 내에서 여러 데이터 품질 검사를 정의할 수 있습니다.

1. 첫 번째 규칙을 Duplicate rows라는 이름으로 만들어 보겠습니다. Rule 1에 대해 아래 옵션을 선택합니다:

- **Data quality check scope**(데이터 품질 검사 범위)에서 **"Individual check for each column"**(각 컬럼에 대해 개별 검사)를 선택합니다.
- **Rule success criteria**에서 **"All data quality checks are met (AND)"** 을 선택합니다.
<!-- 모든 데이터 품질 검사 충족-->
- **Data quality checks**에서 Check 1의 아래 드롭다운하여 **Duplicate rows(중복 행)**을 선택합니다.
- **Condition(조건)에서 Is equals(같음)을 선택합니다.
- **Value에 0**을 입력하고, 드롭다운하여 **rows**을 선택합니다.
- **Rule Summary**에서 설정한 규칙에 대한 설명을 확인할 수 있습니다.
Dataset에 중복 행 수가 == 0인 경우 규칙이 통과됩니다.
    - ![](images/create_sales_dq_ruleset_4.png)

1. **Add another rule**를 클릭하여 dataset에 다른 데이터 품질 검사를 추가하고, 이 규칙의 이름을 *Quantity and total Sales should be >0* 으로 지정해 보겠습니다.

- **Data quality check scope**(데이터 품질 검사 범위)에서 **Common checks for selected columns**(선택한 컬럼에 대한 공통 검사)를 선택합니다.
- **Rule success criteria**(규칙 성공 기준)에서 **All data quality checks are met (AND)** 모든 데이터 품질 검사 충족을 선택합니다.
- **Selected columns**에서 **Selected columns**을 선택합니다.
- **Select Columns**을 클릭하여 **Quantity**와 **Total_Sales**을 두 개를 선택합니다.
- Check 1의 **Data quality check**에서 **Column values** 드롭다운하여 **Numeric values**을 선택합니다.
- **Condition**에서 **Greater than(다음보다 큼)**을 선택합니다.
- **Value**에 **Custom value**으로 *0*을 입력합니다.
- **Threshold(임계값)**의 경우 Condition를 드롭다운하여 **Greater than equals**을 선택, **Threshold(임계값)**을 100, **%(percent) rows** 으로 설정합니다.
- **Rule Summary**에서 설정한 규칙에 대한 설명을 볼 수 있습니다.
Quantity, Total_Sales의 값이 행의 100% 이상에 대해 0 >= 0인 경우 규칙이 통과됩니다.

    - ![](images/create_sales_dq_ruleset_5.png)

1. 이제 데이터 품질 검사를 시작할 준비가 되었습니다. **Create ruleset** 버튼을 클릭하여 데이터 품질 검사를 저장합니다.
     - ![](images/create_sales_dq_ruleset_6.png)


1. 그러면 **DQ RULES**메뉴에 **Data quality rulesets**으로 이동합니다. Sales dataset에 새 규칙 집합을 적용하기 위한 profile Job을 만들기 위해
**Sales DQ Checks**를 선택하고, **Create profile job with ruleset** 를 선택합니다.
     - ![](images/create_sales_dq_ruleset_7.png)


1. job name 텍스트 박스에 *Sales profile*으로 입력하고, **Full dataset**을 선택합니다.
     - ![](images/create_a_profile_3.png)

1.  **Job output settings **의 경우, Browse 버튼을 클릭하여 **glue-databrew-immersionday-xxxx S3 버킷 > profile-output 폴더**를 선택 후 Select 버튼을 클릭합니다.
     - ![](images/create_sales_dq_ruleset_8.png)
1. **Data quality rules** 섹션에서 **Sales DQ Checks** 규칙 집합이 이미 적용되어 있는 것을 볼 수 있습니다.

1. 나머지는 optional 설정은 default settings을 그대로 유지하고, **Role name** 드롭다운하여 *AWSGlueDataBrewServiceRole-ID* role을 선택합니다.

1. **Create and run job**버튼을 클릭합니다.
     - ![](images/create_sales_dq_ruleset_9.png)

Profile jobs 은dataset에 대해 평가를 실행합니다. dataset 수준과 컬럼 수준으로 세분화하여 통계를 생성합니다.

1. 그러면 **Sales Datasets**의 **Data profile overview** 탭으로 이동합니다.
작업이 완료되는 데 약 5분이 소요됩니다.
     - ![](images/verify_result_2.png)

1. **Value distribution(값 분포)**를 확인합니다.
     - ![](images/verify_result_3.png)

1. **Columns statistics** 탭을 선택합니다

     - ![](images/verify_result_4.png)

1. Data quality rules 탭을 선택하면 데이터 품질 검사에 모두 실패한 것을 확인할 수 있습니다.
     - ![](images/create_sales_dq_ruleset_10.png)

1. Advance Transform lab module에서는 데이터 품질 문제가 있는 행을 필터링하기 위해 변환을 수행합니다.
### 2.5 Data Lineage

데이터의 계보를 시각적으로 매핑하여 데이터가 거쳐 온 다양한 데이터 원본과 변환 단계를 파악할 수 있습니다.

1. datasets 메뉴에서 Sales dataset을 선택합니다.

1. **Data lineage** 탭을 선택하여 다음을 확인합니다.
     - ![](images/datalineage.png)

1. 해당 dataset의 모든 작업을 보기위해 **CloudTrail logs**를 선택합니다.
     - ![](images/cloudtrail.png)

## 3. Standard Transform
이 실습에서는 name 컬럼을 표준화 및 결합하고 address 컬럼을 분리하여 customer data를 정리하고 변환합니다.
     - ![](images/basictransform.png)

다음 방법을 실습하게 됩니다.
- Project 만들기
- Recipe 빌드
- Job 생성

### 3.1 Create Project

DataBrew의 대화형 데이터 준비 작업 공간을 project라고 합니다. data project를 사용하여 데이터, 변환 및 예약된 프로세스와 같은 관련 항목 아이템을 관리합니다. 프로젝트 생성의 일부로 작업할 dataset을 선택하거나 만듭니다. 그런 다음, DataBrew가 실행할 일련의 지침 또는 단계인 recipe를 만듭니다. 이러한 작업을 통해 raw data를 분석이나 예측을 위한 데이터 파이프라인에서 사용할 수 있는 형태로 변환합니다.


이제 dataset이 만들어졌으므로 데이터 변환을 시작할 수 있습니다.

1. 왼쪽 메뉴에서 **PROJECTS**를 선택합니다.

1. **Create project**를 선택합니다.

1. 프로젝트 이름을 *CleanCustomer*로 지정합니다. 자동 입력된 Recipe name은 그대로 둡니다.

1. **My datasets**을 선택합니다.

     - ![](images/create_a_project.png)

1. 이전 실습 모듈에서 만든 **Customers dataset**을 선택합니다.

1. **Sampling** 섹션을 열고 Type을 **Random rows**으로 설정합니다.

1. 샘플 크기로 **1,000**을 선택합니다.

     - ![](images/select_dataset.png)

1. Permission 섹션의 **Role name**를 드롭다운에서 *AWSGlueDataBrewServiceRole-ID*를 선택합니다.

1. **Create Project**을 클릭합니다.
     - ![](images/create_iam_role.png)


새 프로젝트를 초기화하는 데 몇 분 정도 걸립니다.


### 3.2 Build Recipe

recipe는 데이터에 대한 일련의 지침 또는 단계로, DataBrew가 작동하도록 하려는 데이터입니다. recipe에는 여러 단계가 포함될 수 있으며, 각 단계에는 여러 작업이 포함될 수 있습니다. toolbar의 transformation 도구를 사용하여 데이터에 적용하려는 모든 변경 사항을 설정할 수 있습니다. DataBrew는 데이터 변환에 대한 지침을 저장하지만 실제 데이터는 저장하지 않습니다. 프로젝트는 기본적으로 데이터 집합의 첫 번째 샘플 n개를 로드합니다. DataBrew는 데이터 집합에 대한 통계를 자동으로 생성하고 데이터의 그리드(샘플 데이터 집합의 표 형식, Excel과 비슷한 시각화), 스키마 및 프로필(전체 dataset에 대한 통계) 보기를 제공합니다. 샘플링을 업데이트하여 변환 프로세스의 어느 시점에서든 작업할 데이터의 다른 부분을 검색할 수 있습니다.

이 실습에서는 병합 및 포맷 변환을 사용하여 이름 컬럼을 표준화합니다. Format transform을 사용하여 생년월일(DoB) 열을 표준화합니다. 다음으로, Clean and SPLIT transform을 사용하여 Address 컬럼을 표준화합니다. 마지막으로 최종 출력에 있는 PII 데이터를 삭제합니다.

1. 이전 단계에서 만든 CleanCustomer 프로젝트를 엽니다.

1. 상단 메뉴에서 MERGE을 선택합니다.

     - ![](images/build_recipe_1.png)

1. Source 컬럼으로 Prefix, First_Name 및 Last_Name 컬럼을 선택합니다.

1. 공백 문자를 separator(구분 기호)로 입력합니다.
     - ![](images/build_recipe_2.png)

1. **New column name** 텍스트 상자에 `#0969DA` Name 을 입력합니다.

1. **Preview changes**를 클릭하고, 미리 보기에서 예상한 결과가 표시되는지 확인합니다. **Apply**를 선택합니다.

# AWS CloudFormation

## CloudFormation 개요

CloudFormation은 AWS에서 제공하는 인프라 자동화 도구로, 인프라 리소스를 코드 형태로 정의하고 관리할 수 있도록 도와주는 서비스입니다. 이를 통해 EC2, S3, RDS, IAM 등 다양한 AWS 리소스를 선언형 템플릿으로 정의하여 일괄적으로 배포하고 관리할 수 있습니다.

CloudFormation은 YAML 또는 JSON 템플릿 파일을 사용하여 리소스를 정의하며, 이를 실행하면 스택(Stack)이 생성됩니다. 하나의 스택은 템플릿에 정의된 리소스들의 모음이며, 이를 통해 반복적이고 일관된 인프라 구성이 가능합니다. 이 방식은 수작업으로 리소스를 생성하는 것보다 오류를 줄이고, 관리 및 변경이 용이합니다.

## CloudFormation 스택 생성

스택(Stack)은 CloudFormation 템플릿의 실행 단위입니다. 스택을 생성할 때는 템플릿 경로(로컬 파일 또는 S3), 매개변수 값(Parameter), IAM 역할 등을 지정해야 합니다. 콘솔, CLI, SDK 등 다양한 방법으로 스택을 생성할 수 있습니다.

스택 생성 시 CloudFormation은 템플릿의 Resources 섹션에 정의된 각 리소스를 순서대로 생성하며, 필요한 경우 의존성에 따라 생성 순서를 자동 조정합니다.

## CloudFormation 업데이트 및 스택 삭제

CloudFormation은 기존 스택을 수정하거나 확장할 수 있는 업데이트 기능을 제공합니다. 템플릿 수정 후 다시 실행하면 CloudFormation은 변경된 부분만 식별하여 해당 리소스만 업데이트합니다.

스택을 삭제하면 관련된 모든 리소스가 삭제되며, 예외적으로 삭제 정책(DeletePolicy)이 설정된 리소스는 보존하거나 스냅샷이 생성될 수 있습니다.

## YAML 단기 집중

YAML은 CloudFormation 템플릿에서 가장 널리 사용되는 포맷입니다. JSON도 지원되지만, YAML은 구조가 간결하고 들여쓰기를 기반으로 가독성이 뛰어나 실무에서 선호됩니다. 또한 CloudFormation의 내장 함수 사용 시 YAML은 더 직관적으로 작성할 수 있습니다.

예를 들어 `!Ref`, `!Sub`, `!GetAtt` 같은 내장 함수는 YAML에서 접두사로 사용되어, 긴 함수 호출 대신 간단한 표기로 처리할 수 있어 유지보수가 편리합니다.

## CloudFormation 리소스

Resources 섹션은 CloudFormation 템플릿의 핵심 구성 요소입니다. 여기에 선언된 리소스가 실제로 AWS에 생성됩니다. 각 리소스는 논리 ID(Logical ID), 리소스 타입(Type), 속성(Properties)으로 구성됩니다.

예를 들어 다음과 같은 리소스를 정의할 수 있습니다:

```yaml
Resources:
  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-unique-bucket-name
```

이처럼 S3 버킷, EC2 인스턴스, IAM 역할 등 다양한 리소스를 선언적으로 정의할 수 있습니다.

## CloudFormation 매개변수

Parameters 섹션은 템플릿 외부에서 값을 주입받기 위해 사용합니다. 매개변수를 사용하면 템플릿을 재사용 가능하게 만들 수 있으며, 다양한 환경(dev, test, prod) 간 구성을 유연하게 조절할 수 있습니다.

매개변수는 Type, Default, AllowedValues, Description 등을 통해 유효성 검사를 적용할 수 있습니다.

```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t3.micro
    Description: EC2 인스턴스 타입 선택
```

## CloudFormation 매핑

Mappings 섹션은 지역별 또는 조건별 값을 매핑하는 정적 테이블입니다. 보통 AWS 리전별 AMI ID를 정의할 때 자주 사용됩니다.

```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0abcdef1234567890
    ap-northeast-2:
      AMI: ami-0fedcba9876543210
```

Mappings 값은 `Fn::FindInMap` 또는 `!FindInMap` 함수로 참조할 수 있습니다. 매개변수나 조건에 따라 값을 동적으로 선택하는 데 활용됩니다.
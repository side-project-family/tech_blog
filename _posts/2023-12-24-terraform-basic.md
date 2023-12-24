---
title: 테라폼 설치와 AWS 사용해보기
author: 종호
date: 2023-12-24 12:10:00 +0900
categories: [Terraform, AWS, S3, API, Lambda]
tags: [writing]
render_with_liquid: false
---

>  테라폼의 설치 및 환경구성하며, AWS Provider 설정과 API GW, Lambda, S3 연동을 다루는 글 입니다.
{: .prompt-info }

## Terraform 이란 ?
테라폼은 인프라스트럭처를 코드(IaC)로 관리하는 도구로, 클라우드 서비스 및 온프레미스 인프라를 효과적으로 배포하고 관리할 수 있게 해줍니다. 설정 파일을 통해 원하는 인프라 상태를 정의하고, 이를 통해 테라폼이 자동으로 배포 및 관리 작업을 수행합니다.

## IaC 란 ?
 IaC는 "Infrastructure as Code"의 약어로, 인프라스트럭처를 코드로 정의하고 관리하는 개념을 나타냅니다. IaC는 전통적인 수동으로 수행되는 인프라 관리 작업을 자동화하고, 코드로 인프라를 버전 관리하고 추적함으로써 효율성과 일관성을 높이는 방법을 제공합니다.

여러 가지 클라우드 서비스와 온프레미스 인프라를 효과적으로 관리하기 위해 IaC를 사용할 수 있습니다. 일반적으로 IaC는 다음과 같은 특징을 갖습니다:

- 선언적 언어 사용: IaC는 인프라를 어떻게 구성할지를 선언적으로 정의합니다. 사용자는 목표 상태를 기술하고, IaC 도구는 해당 상태를 달성하도록 작업을 수행합니다.

- 자동화: IaC는 인프라 구성 및 관리 작업을 자동화하여 반복적이고 수동적인 작업을 최소화합니다. 이는 시스템 배포, 구성 변경, 확장, 축소 등의 작업에 적용됩니다.

- 상태 관리: IaC 도구는 인프라의 현재 상태를 추적하고, 변경 사항을 추적할 수 있도록 합니다. 이를 통해 변경 이력을 확인하고, 롤백이나 비교 등의 작업이 가능해집니다.

- 멀티클라우드 및 하이브리드 클라우드 환경 지원: 다양한 클라우드 제공업체와 온프레미스 환경에서도 IaC를 사용하여 일관된 방식으로 인프라를 관리할 수 있습니다.

- 버전 관리: IaC 코드는 버전 관리 시스템(Git, SVN 등)을 활용하여 코드 변경 이력을 관리할 수 있습니다.

- 테스트와 검증: IaC 코드는 테스트와 검증을 통해 안정적인 인프라를 보장할 수 있습니다. 코드 내 오류를 사전에 감지하고 테스트를 통해 예상치 못한 문제를 방지할 수 있습니다.

주요 IaC 도구로는 Terraform, AWS CloudFormation, Ansible, Chef, Puppet 등이 있습니다. 이러한 도구들은 각자의 특징과 문법을 가지고 있으며, 선택은 프로젝트의 요구사항과 개발자 선호에 따라 다를 수 있습니다.

## Terraform 설치

[공식 사이트 설치 페이지](https://developer.hashicorp.com/terraform/install)

**mac OS**
```shell
$ brew tap hashicorp/tap
$ brew install hashicorp/tap/terraform
```

**Windows OS**

- [Binary Downoad for 386](https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_windows_386.zip)

- [Binary Downoad for AMD64](https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_windows_amd64.zip)

**Linux(Ubunto)**
```shell
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | 
sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

## Terrafrom 파일 생성
.tf 파일은 Terraform에서 사용되는 설정 파일로, Terraform에게 인프라스트럭처를 코드로 정의하고 관리하는 데 사용됩니다. 이 파일들은 HCL (HashiCorp Configuration Language) 형식으로 작성되며, 특정 인프라스트럭처 리소스와 설정을 정의하는 데 사용됩니다.

**HLLC 이란?**

HCL(HashiCorp Configuration Language)은 HashiCorp에서 개발한 구성 언어로, 주로 인프라스트럭처와 관련된 설정 파일을 작성하는 데 사용됩니다. Terraform, Vault, Consul 등 HashiCorp의 제품에서 사용되며, 간결하면서도 가독성이 뛰어나고 기계가 읽기 쉬운 형식을 제공하는 것이 특징입니다.

HCL은 JSON이나 YAML과 같은 다른 데이터 구조 표현보다 사람이 읽고 쓰기 쉽도록 설계되었습니다. 

주요 특징
- 간결하고 가독성이 좋음
- 선언적인 문법
- 변수 및 함수 지원
- 주석 지원

## AWS Provider 연결

```hcl
provider "aws" {
  region = "us-east-1"
  access_key = "YOUR_ACCESS_KEY"
  secret_key = "YOUR_SECRET_KEY"
}
```

## S3에 업로드 해보기

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "example_bucket" {
  bucket = "your-unique-bucket-name"
  acl    = "private"
}

resource "aws_s3_bucket_object" "example_object" {
  bucket = aws_s3_bucket.example_bucket.bucket
  key    = "example.txt"
  source = "local/path/to/example.txt" # 실제 로컬 파일 경로로 변경
  acl    = "private"
}
```

## Lambda 업로드 해보기

Lambda 코드 작성

index.js
```javascript
exports.handler = async (event) => {
    console.log('Hello, Lambda!');
    return {
        statusCode: 200,
        body: JSON.stringify('Hello from Lambda!'),
    };
};
```

```
provider "aws" {
  region = "us-east-1"
}

resource "aws_lambda_function" "example_lambda" {
  function_name = "example_lambda_function"
  handler      = "index.handler"
  runtime      = "nodejs14.x"
  filename     = "lambda_function.zip"
  
  role = aws_iam_role.lambda_exec.arn

  source_code_hash = filebase64("lambda_function.zip")

  environment {
    variables = {
      key1 = "value1"
      key2 = "value2"
    }
  }
}

resource "aws_iam_role" "lambda_exec" {
  name = "example_lambda_role"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      }
    }
  ]
}
EOF
}
```


## API GW 연결 해보기

```
provider "aws" {
  region = "us-east-1"
}

resource "aws_lambda_function" "example_lambda" {
  function_name = "example_lambda_function"
  handler      = "index.handler"
  runtime      = "nodejs14.x"
  filename     = "lambda_function.zip"
  
  role = aws_iam_role.lambda_exec.arn

  source_code_hash = filebase64("lambda_function.zip")

  environment {
    variables = {
      key1 = "value1"
      key2 = "value2"
    }
  }
}

resource "aws_iam_role" "lambda_exec" {
  name = "example_lambda_role"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      }
    }
  ]
}
EOF
}

resource "aws_api_gateway_rest_api" "example_api" {
  name        = "example_api"
  description = "Example API"
}

resource "aws_api_gateway_resource" "example_resource" {
  rest_api_id = aws_api_gateway_rest_api.example_api.id
  parent_id   = aws_api_gateway_rest_api.example_api.root_resource_id
  path_part   = "example"
}

resource "aws_api_gateway_method" "example_method" {
  rest_api_id   = aws_api_gateway_rest_api.example_api.id
  resource_id   = aws_api_gateway_resource.example_resource.id
  http_method   = "POST"
  authorization = "NONE"
}

resource "aws_api_gateway_integration" "example_integration" {
  rest_api_id = aws_api_gateway_rest_api.example_api.id
  resource_id = aws_api_gateway_resource.example_resource.id
  http_method = aws_api_gateway_method.example_method.http_method

  integration_http_method = "POST"
  type                    = "AWS_PROXY"
  uri                     = aws_lambda_function.example_lambda.invoke_arn
}

resource "aws_api_gateway_method_response" "example_method_response" {
  rest_api_id = aws_api_gateway_rest_api.example_api.id
  resource_id = aws_api_gateway_resource.example_resource.id
  http_method = aws_api_gateway_method.example_method.http_method
  status_code = "200"

  response_parameters = {
    "method.response.header.Content-Type" = "'application/json'"
  }
}

resource "aws_api_gateway_deployment" "example_deployment" {
  depends_on = [
    aws_api_gateway_integration.example_integration,
  ]

  rest_api_id = aws_api_gateway_rest_api.example_api.id
  stage_name  = "prod"
}
```

## 마치며

> 추가적인 질문은 side-project-family@gmail.com 으로 주세요 :D

## 참고 링크

| Document URL                                                                                          | Description   | 
| -------------------------------------------------------------------------------------------------- | ---------- | 
| [https://developer.hashicorp.com/terraform](https://developer.hashicorp.com/terraform) | `terraform guide` |

---
title: 테라폼 설치와 AWS 사용해보기
author: 종호
date: 2024-1-3 20:25:00 +0900
categories: [Terraform, AWS, S3, IaC]
tags: [writing]
render_with_liquid: false
---

>  테라폼의 설치 및 환경구성하며, AWS Provider 설정 S3 정적페이지 생성을 다루는 글 입니다.
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

> Apple Chip을 사용하는 경우는 brew 설치 시 아래 설정을 추가해주세요. [user]에는 현재 유저의 정보를 입력합니다.
{: .prompt-tip }

```shell
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/[user]/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
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

**HLC 이란?**
HCL(HashiCorp Configuration Language)은 HashiCorp에서 개발한 구성 언어로, 주로 인프라스트럭처와 관련된 설정 파일을 작성하는 데 사용됩니다. Terraform, Vault, Consul 등 HashiCorp의 제품에서 사용되며, 간결하면서도 가독성이 뛰어나고 기계가 읽기 쉬운 형식을 제공하는 것이 특징입니다.

HCL은 JSON이나 YAML과 같은 다른 데이터 구조 표현보다 사람이 읽고 쓰기 쉽도록 설계되었습니다. 

주요 특징
- 간결하고 가독성이 좋음
- 선언적인 문법
- 변수 및 함수 지원
- 주석 지원

## AWS Provider 연결

```
provider "aws" {
  region = "ap-northeast-2" # 서울 리전 코드
  access_key = "YOUR_ACCESS_KEY" # 권장하지 않음
  secret_key = "YOUR_SECRET_KEY" # 권장하지 않음
}
```

> AWS 프로바이더 키를 등록하는 방법은 세 가지가 있습니다. 첫 번째로 환경 변수를 사용하는 방법이 있습니다. 두 번쨰로 AWS CLI를 사용하여 키와 비밀 키를 입력해두면 terraform init에서 읽어오게 됩니다. 이 방법을 가장 많이 사용합니다. 마지막 방법으로는 terraform 코드에 직접 입력하는 방법입니다. 프로바이더 정의시 키를 입력합니다. 이 방법은 보안이슈로 권장하지않습니다.
{: .prompt-tip }

## S3 버켓 생성하기
provider "aws" {
  region = "ap-northeast-2"
}

"aws_s3_bucket"는 클라우드 프로바이더가 제공하는 서비스의 이름이며, "side_project_family_static_web"은 생성되는 서비스의 이름 입니다. (표시되는 이름 x)

```
resource "aws_s3_bucket" "side_project_family_static_web" {
  bucket = "sf.static.web" # 실제 생성되는 버켓의 이름

  tags = {
      Name = "side-project-family-web-bucket" # 버켓의 태그 이름
  }

  website {
    index_document = "index.html" # index page 설정
    error_document = "index.html" # error page 설정
  }
}
```

## ACL(Access Control List) 설정
ACL(Access Control List)은 버킷 및 객체(파일)에 대한 액세스 권한을 관리하는 데 사용되는 설정입니다. 각각의 S3 버킷과 객체에는 고유한 ACL이 적용됩니다.
S3의 ACL은 주로 다음과 같은 권한을 관리합니다.

버킷 수준 ACL:
- Private (private): 기본값으로, 버킷의 모든 객체에 대한 액세스가 제한됩니다.
- Public Read (public-read): 버킷의 모든 객체에 대한 읽기 권한이 모든 사용자에게 부여됩니다.
- Public Read/Write (public-read-write): 버킷의 모든 객체에 대한 읽기 및 쓰기 권한이 모든 사용자에게 부여됩니다.

객체 수준 ACL:
- Private (private): 기본값으로, 객체에 대한 액세스가 제한됩니다.
- Public Read (public-read): 객체에 대한 읽기 권한이 모든 사용자에게 부여됩니다.
- Public Read/Write (public-read-write): 객체에 대한 읽기 및 쓰기 권한이 모든 사용자에게 부여됩니다.

ACL은 특정 사용자, 그룹, 혹은 모든 사용자에 대한 권한을 부여하거나 거부하는 방식으로 설정됩니다. 예를 들어, Canonical User ID를 사용하여 특정 AWS 계정에 대한 액세스를 지정하거나, AllUsers 그룹을 사용하여 모든 사용자에게 액세스를 부여할 수 있습니다.

Terraform을 사용하면 aws_s3_bucket 및 aws_s3_bucket_object 리소스에서 acl 속성을 설정하여 S3 버킷 및 객체의 ACL을 정의할 수 있습니다.

```
resource "aws_s3_bucket_ownership_controls" "side_project_family_static_web" {
  bucket = "sf.static.web"
  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

resource "aws_s3_bucket_public_access_block" "side_project_family_static_web" {
  bucket = "sf.static.web"

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_acl" "side_project_family_static_web" {
  depends_on = [
    aws_s3_bucket_ownership_controls."side_project_family_static_web",
    aws_s3_bucket_public_access_block."side_project_family_static_web",
  ]

  bucket = "sf.static.web"
  acl    = "public-read"
}
```

## S3 bucket policy
정적페이지로 호스팅을 하기위해서 퍼블릭환경에서 접속할 수 있는 정책을 추가합니다.

```
resource "aws_s3_bucket_policy" "bucket-policy" {
  bucket = "sf.static.web"

  policy = <<POLICY
{
  "Id": "Policy",
  "Statement": [
    {
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::sf.static.web/*", # 주의: 사용자의 버켓 이름을 꼭 넣어주세요 !
      "Principal": {
        "AWS": [
          "*"
        ]
      }
    }
  ]
}
POLICY
}
```

## s3 url
인프라 적용 후 버켓으로 바로 이동가능하도록 url 출력

```
output "side_project_family_static_web_url" {
  value = "http://sf.static.web.s3-website.ap-northeast-2.amazonaws.com"
}
```

## 마치며

> 추가적인 질문은 side-project-family@gmail.com 으로 주세요 :D

## 참고 링크

| Document URL                                                                                          | Description   | 
| -------------------------------------------------------------------------------------------------- | ---------- | 
| [https://developer.hashicorp.com/terraform](https://developer.hashicorp.com/terraform) | `terraform guide` |

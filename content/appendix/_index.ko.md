---
title: "Appendix"
weight: 900
pre: "<b>7. </b>"
---

{{% notice note %}}
이 실습을 마치면 사용한 AWS 계정에 비용이 추가로 발생하지 않도록 사용한 리소스를 삭제해야 합니다.
{{% /notice %}}

AWS CDK를 이용해서 배포하는 방법을 소개 합니다.

### Prerequisites
1. AWS CDK Toolkit을 설치합니다.

    ```shell script
    npm install -g aws-cdk
    ```

2. cdk가 정상적으로 설치되었는지, 다음 명령어를 실행해서 확인합니다.
    ```
    cdk --version
    ```
   예)
    ```shell script
    $ cdk --version
    1.71.0 (build 953bc25)
    ```

##### Useful commands

 * `cdk ls`          list all stacks in the app
 * `cdk synth`       emits the synthesized CloudFormation template
 * `cdk deploy`      deploy this stack to your default AWS account/region
 * `cdk diff`        compare deployed stack with current state
 * `cdk docs`        open CDK documentation

### Deployment

CDK로 배포할 경우, 아래 아키텍처 그림의 `1(a), 1(b), 1(c), 1(f), 2(b), 2(a)`가 자동으로 생성됩니다.

![aws-analytics-system-build-steps-extra](/analytics-on-aws/images/aws-analytics-system-build-steps-extra.svg)

1. [Getting Started With the AWS CDK](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html)를 참고해서 cdk를 설치하고,
cdk를 실행할 때 사용할 IAM User를 생성한 후, `~/.aws/config`에 등록합니다.
예를 들어서, cdk_user라는 IAM User를 생성 한 후, 아래와 같이 `~/.aws/config`에 추가로 등록합니다.

    ```shell script
    $ cat ~/.aws/config
    [profile cdk_user]
    aws_access_key_id=AKIAI44QH8DHBEXAMPLE
    aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
    region=us-east-1
    ```

2. Lambda Layer에 등록할 Python 패키지를 생성한 뒤 S3 버킷에 업로드 합니다. [이 링크](https://serverless-bi-system-from-scratch.workshop.aws/ko/reference.html)의 절차를 따라 수행하십시오. 
    <!--3. 생성된 Python 패키지를 업로드 할 s3 bucket을 생성합니다.`lambda-layer-resources-xxxxxxxx`라는 이름의 s3 bucket을 생성합니다. `xxxxxxxx` 는 다른 s3 bucket 이름이 겹치지 않도록 임의의 숫자나 문자를 입력 합니다. 아래 스크립트의 xxxxxxxx 부분을 입력한 임의의 값으로 대체하십시오.

    ```
    aws s3 mb s3://lambda-layer-resources-xxxxxxxx

    aws s3 cp es-lib.zip s3://lambda-layer-resources-<<USERNAME>>/var/ # 압축한 패키지를 s3에 업로드 한 후, lambda layer에 패키지를 등록할 때, s3 위치를 등록하면 됨

    ```
    ```shell script
    $ aws s3 ls s3://lambda-layer-resources/var/
    2019-10-25 08:38:50          0
    2019-10-25 08:40:28    1294387 es-lib.zip
    ``` -->

1. ES 도메인을 최초 생성하는 경우, [Service Linked Role](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/slr-es.html) 을 생성해주어야 합니다. 아래 명령어를 수행하여 생성하십시오. 
    ```
    aws iam create-service-linked-role --aws-service-name es.amazonaws.com 
    ```
    이 과정이 사전에 수행되지 않으면 아래와 같은 에러가 발생하면서 cdk 스택 배포가 실패하게 됩니다.

     ```
      Search Before you can proceed, you must enable a service-linked role to give Amazon ES permissions to access your VPC. (Service: AWSElasticsearch; Status Code: 400; Error Code: ValidationException;)
     ```

2. 소스 코드를 git에서 다운로드 받은 후, `S3_BUCKET_LAMBDA_LAYER_LIB` 라는 환경 변수에 lambda layer에 등록할 패키지가 저장된 s3 bucket 이름을
설정 한 후, `cdk deploy` 명령어를 이용해서 배포합니다. 아래 

    ```shell script
    $ git clone https://github.com/ksmin23/aws-analytics-immersion-day.git
    $ cd aws-analytics-immersion-day
    $ python3 -m venv .env
    $ source .env/bin/activate
    (.env) $ pip install -r requirements.txt
    (.env) $ export CDK_DEFAULT_ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
    (.env) $ export CDK_DEFAULT_REGION=us-west-2
    (.env) $ cdk bootstrap aws://${CDK_DEFAULT_ACCOUNT}/${CDK_DEFAULT_REGION}
    (.env) $ export S3_BUCKET_LAMBDA_LAYER_LIB=lambda-layer-resources
    (.env) $ cdk --profile cdk_user deploy
    ```

   :white_check_mark: `cdk bootstrap ...` 명령어는 CDK toolkit stack 배포를 위해 최초 한번만 실행 하고, 이후에 배포할 때는 CDK toolkit stack 배포 없이 `cdk deploy` 명령어만 수행하면 됩니다.

    ```shell script
    (.env) $ export CDK_DEFAULT_ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
    (.env) $ export CDK_DEFAULT_REGION=us-west-2
    (.env) $ export S3_BUCKET_LAMBDA_LAYER_LIB=lambda-layer-resources
    (.env) $ cdk --profile cdk_user deploy
    ```

1. 배포한 애플리케이션을 삭제하려면, `cdk destroy` 명령어를 아래와 같이 실행 합니다.
    ```shell script
    (.env) $ cdk --profile cdk_user destroy
    ```


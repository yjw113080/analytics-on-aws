---
title: "Verify"
weight: 33
pre: "<b>3-3. </b>"
---

![aws-analytics-system-build-steps](/analytics-on-aws/images/aws-analytics-system-build-steps.png)

## 데이터 파이프라인 동작 확인 하기

샘플 데이터를 이용해서 `Kinesis Data Streams -> Kinesis Data Firehose -> S3` 로 데이터가 정상적으로 수집되는지 확인합니다.

1. 앞서 생성한 E2 인스턴스에 SSH 접속을 합니다.
2. `gen_kinesis_data.py`을 실행합니다.
    ```shell script
    python3 gen_kinesis_data.py -I resources/online_retail.csv \
    --service-name kinesis \
    --out-format json \
    --stream-name retail-trans
    ```
1. 매 초 데이터가 발생하는 것을 확인합니다. 충분한 데이터 수집을 위해 실행 중인 상태로 다음 단계를 진행합니다.

    ![gen_kinesis_data](/analytics-on-aws/images/gen_kinesis_data.png)

2. 몇 분 뒤 생성한 S3 bucket을 확인해 보면, 생성된 원본 데이터가 Kinesis Data Firehose를 통해 S3에 저장되는 것을 확인할 수 있습니다.

    ![firehose-result](/analytics-on-aws/images/firehose-result.png)
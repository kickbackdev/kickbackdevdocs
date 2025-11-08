### Test Scenarios

| Case ID | Template Name                         | Test Time      | Status          | Airtable Record Created?              | Slack Alert Received? | Notes                                                  |
| ------- | ------------------------------------- | -------------- | --------------- | ------------------------------------- | --------------------- | ------------------------------------------------------ |
| S1      | Kickback - Individual Parts Agreement | 25.11.08 15:55 | Success         | Partners (ID: …)<br>Contracts (ID: …) | #automation-alerts    | Partner Type = Individual 확인                           |
| S2      | Kickback - Shop Partner Agreement     | 25.11.08 15:30 | Success         | Partners (ID: …)<br>Contracts (ID: …) | #automation-alerts    | Shop Type = Charter 및 Charter Status End Date = +1y 확인 |
| S3      | Updated Ambassador Agreement          | 25.11.08 15:37 | Success         | Partners (ID: …)<br>Contracts (ID: …) | #automation-alerts    | Partner Type = Ambassador 확인                           |
| S4      | Kickback - Individual Parts Agreement | 15.11.08 15:20 | Success         | Parts (ID: …)                         | #automation-alerts    | Owner가 S1 파트너와 연결되고 Condition = Like New 확인            |
| S5      | Kickback - Individual Parts Agreement | 25.11.08 16:04 | Success (Logic) | N/A (생성 안 됨)                          | #ops-alerts           | ghost+dev 파트너 미존재로 정상 중단 및 경고 확인 (음수 테스트 성공)           |

---

### Example Partner Records

| Partner ID     | Partner name              | Partner Type | Email      | Stripe Connected Account ID                         | Stripe Customer ID | SignWell Document ID | Referrals                            | Referred By | Shop Type | Subscription Status | Charter Status End Date | Stripe Account Status | Sales | Payouts | Contracts                            | Referral Role |
| -------------- | ------------------------- | ------------ | ---------- | --------------------------------------------------- | ------------------ | -------------------- | ------------------------------------ | ----------- | --------- | ------------------- | ----------------------- | --------------------- | ----- | ------- | ------------------------------------ | ------------- |
| S1 Test Record | yuhyun2-rechck6AnO63MQNQq | yuhyun2      | Individual | [top6543top@naver.com](mailto:top6543top@naver.com) |                    |                      | 79466d25-3083-4eab-a8f4-f74462130a81 |             |           | UNMAPPED            | Active                  |                       |       |         | 79466d25-3083-4eab-a8f4-f74462130a81 | N/A           |
| S2 Test Record | yuhyun2-rec8VlKQdU5o46md9 | yuhyun2      | Shop       | [top6543top@naver.com](mailto:top6543top@naver.com) |                    |                      | a4c90874-48d5-463a-8872-0573ca2b0c11 |             |           | Charter             | Active                  |                       |       |         | a4c90874-48d5-463a-8872-0573ca2b0c11 | N/A           |
| S3 Test Record | yuhyun2-reczqptteoc5Mbs0w | yuhyun2      | Ambassador | [top6543top@naver.com](mailto:top6543top@naver.com) |                    |                      | fb06f4ee-89b9-4e68-996f-9b6a1b69c4a9 |             |           |                     | Active                  |                       |       |         | fb06f4ee-89b9-4e68-996f-9b6a1b69c4a9 | N/A           |

---

### Example Contract Records

| Contract ID        | Partner                              | Contract Type             | Date Signed | Document Link   |                                                                                                                                            |
| ------------------ | ------------------------------------ | ------------------------- | ----------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| S1 Contract Record | 79466d25-3083-4eab-a8f4-f74462130a81 | yuhyun2-rechck6AnO63MQNQq | Individual  | 2025.11.8 06:55 | [https://www.signwell.com/app/signed/WVPVnbAAx7SY4Ppx3Z27txExMjD.pdf](https://www.signwell.com/app/signed/WVPVnbAAx7SY4Ppx3Z27txExMjD.pdf) |

---

### Example Part Records

| Part ID        | QR Code ID | Item Name | Owner                     | Status                             | Part Description | Vehicle Compatibility | Condition | A Location Code | Date Added | Sales | Sale record |
| -------------- | ---------- | --------- | ------------------------- | ---------------------------------- | ---------------- | --------------------- | --------- | --------------- | ---------- | ----- | ----------- |
| S4 Test Record | 42         |           | yuhyun2-rec7VP36rBhiia2sa | Contract Signed - Awaiting Receipt | 1                | 2019 bmw x4           | New       |                 | 2025.11.8  |       |             |

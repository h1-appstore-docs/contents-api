# H1 App Store Contents API

## 목차

1. [소개](#1-소개)
2. [유통사 조회](#2-유통사-조회)
3. [상품 조회](#3-상품-조회)
4. [유효성 조회](#4-유효성-조회)
5. [결제 요청](#5-결제-요청)
6. [결제 내역 조회](#6-결제-내역-조회)

### 1. 소개

본 가이드는 H1 App Store에 등록된 상품 목록을 조회하고, 유효기간을 확인하고, 결제를 요청하는 방법에 대해서 안내합니다.  
모든 API는 표준 HTTP 통신을 이용합니다. HTTP 통신은 현대 프로그래밍에서는 표준화된 방법이므로 Java, C, Javascript, Python 등 모든 언어와 플랫폼에서 사용할 수 있습니다.  
본 가이드에서는 가장 표준화된 방식인 curl 코드를 제공해 사용하는 환경에 맞추어 개발할 수 있도록 가이드를 제공합니다.

#### Base URL

모든 API는 다음 호스트를 이용해 호출합니다.

```shell script
https://h1appstore.hancom-toki.com:9443
```

#### 요청 Request

모든 요청은 웹 표준 요청 형식을 따릅니다.  
GET 요청의 경우 `querystring` 형식으로 작성합니다.  
POST 요청은 `application/json` 형식을 기본으로 합니다.  

#### 응답 Response

모든 응답은 JSON 형식입니다. 

### 2. 유통사 조회
#### [GET] /devices/{SERIAL_NUMBER}/distributors

가장 먼저 할 일은 토키의 시리얼 넘버를 획득하는 것입니다.  
획득한 시리얼 넘버를 바탕으로 유통사의 정보를 조회할 수 있습니다.

본 요청은 쿼리스트링 없이 path 내에 시리얼 넘버가 전달되는 것에 유의하세요.

```shell script
curl -X GET "https://h1appstore.hancom-toki.com:9443/api/toki/devices/{SERIAL_NUMBER}/distributors" -H "accept: */*"
```  

시리얼 넘버가 유효하다면 다음과 같은 응답을 받을 수 있습니다.

```json
{
  "id": 1,
  "name": "유통사명",
  "status": "정상",
  "secretKey": "750ff6f6-01f0-pqow-zmxn-c40aa28b53a1",
  "businessNumber": "000-00-00000",
  "ceo": "대표자명",
  "agent": "담당자명",
  "contact": "02-000-0000",
  "email": "email@email.com",
  "mobile": "010-0000-0000",
  "zipcode": "07803",
  "address": "서울 강서구 강서로 375",
  "addressDetail": "0동 0호",
  "logo": null,
  "memo": null,
  "createdAt": null,
  "updatedAt": "2020-11-02T06:34:08.000+00:00",
  "products": [
    "..."
  ],
  "devices": [
    "..."
  ]
}
```

응답에서 `id`와 `secretKey`를 사용해 다음 진행으로 넘어갑니다.

### 3. 상품 조회
#### [GET] /api/toki/distributors/{ID}/products

|파라미터명    |필수    |기본값|설명                  |
|------------|-------|-----|---------------------|
|secretKey   |**예** |     |유통사의 Secret Key    |
|appName     |아니오  |     |상품의 앱 이름으로 검색  |
|page        |아니오  |0    |페이지네이션 위치       |

유통사 조회를 통해 얻은 `id`와 `secretKey`를 이용해 등록된 상품을 조회합니다. `id`는 path 상에 들어가고 `secretKey`는 쿼리 스트링으로 전달되는 것에 유의하십시오.  
한 페이지당 10개의 항목이 반환되므로 추가 로드를 위해서는 `page` 키를 이용해 페이징 해야 합니다. 페이징 횟수를 조절하기 위해 미리 `appName`을 이용해 검색 결과를 필터할 수 있습니다.

```shell script
curl -X GET "https://h1appstore.hancom-toki.com:9443/api/toki/distributors/{ID}/products?page=0&secretKey={secretKey}" -H "accept: */*"
```

요청이 성공하면 다음과 같은 응답을 받을 수 있습니다.

```json
{
  "content": [
    {
      "id": 123,
      "distributorId": 1,
      "free": false,
      "availability": true,
      "usagePeriod": 1,
      "product": {
        "id": 100,
        "name": "상품명",
        "thumbnail": "https://image.url/file.ext",
        "appPrice": 1000,
        "textbookPrice": 1000,
        "...기타 필드...": "...표시 생략..."
      },
      "createdAt": null,
      "updatedAt": "2020-10-30T06:52:29.000+00:00"
    },
    "...반복..."
  ],
  "pageable": {
    "sort": {
      "sorted": false,
      "unsorted": true,
      "empty": true
    },
    "pageNumber": 0,
    "pageSize": 10,
    "offset": 0,
    "unpaged": false,
    "paged": true
  },
  "last": true,
  "totalPages": 1,
  "totalElements": 6,
  "first": true,
  "sort": {
    "sorted": false,
    "unsorted": true,
    "empty": true
  },
  "numberOfElements": 6,
  "size": 10,
  "number": 0,
  "empty": false
}
```

`content`는 상품과 상품의 정책이 포함된 배열입니다.  
`content`의 요소 내에는 사용기간, 유효기간 등의 정책 정보가 있고, 상품에 대한 실 표시 정보는 `product` 라는 자식 오브젝트로 표시된 것에 유의하십시오.  
상품의 표지 이미지나 상품명, 가격 등의 정보는 `product` 내에서 확인할 수 있습니다.

### 4. 유효성 조회
#### [GET] /api/toki/distributors/{distributorId}/products/{productId}

|파라미터명    |필수    |기본값|설명                  |
|------------|-------|-----|---------------------|
|serial   |**예** |     |토키의 시리얼 넘버    |

상품의 목록을 얻었다면 현재 사용중인 토키가 해당 상품에 대한 재생 권한이 있는지 확인이 필요할 것입니다.  
상품이 무료인지, 유료인지, 유료라면 결제 기록이 있는지, 유효기간 범위 내에 있는지 등을 확인하고 성공/실패로 응답을 줍니다.  
path 내의 `distributorId`에는 유통사의 id를, `productId`에는 상품의 id를 넣으세요.  
기기의 시리얼 넘버는 쿼리스트링으로 전달해야합니다.

```shell script
curl -X GET "https://h1appstore.hancom-toki.com:9443/api/toki/distributors/{distributorid}/products/{productId}?serial={serialKey}" -H "accept: */*"
```

컨텐츠 재생이 유효할 시에는 HTTP 200 OK 응답을 확인할 수 있습니다.

### 5. 결제 요청
#### [POST] /api/toki/orders

결제는 하나의 상품에 대해서 이루어 질 수도 있고, 여러개의 상품으로 이루어질 수도 있습니다.  
여러개의 상품을 묶어서 결제를 하더라도 결제는 1회만 이루어져야 하므로, 요청은 배열을 기본으로 합니다.

다음의 요청 예시를 보십시오.

```json
{
  "buyerName": "구매자 이름",
  "buyerTel": "구매자 연락처",
  "approvalDTO": [
    { "distributorId": 1, "productId": 123 },
    { "distributorId": 1, "productId": 124 },
    { "distributorId": 1, "productId": 125 },
    { "distributorId": 1, "productId": 126 }
  ],
  "serial": "토키 시리얼 넘버"
}
```

```shell script
curl -X POST "https://h1appstore.hancom-toki.com:9443/api/toki/orders" -H "accept: */*" -H "Content-Type: application/json" -d "{\"buyerName\":\"구매자 이름\",\"buyerTel\":\"구매자 연락처\",\"approvalDTO\":[{\"distributorId\":1,\"productId\":123},{\"distributorId\":1,\"productId\":124},{\"distributorId\":1,\"productId\":125},{\"distributorId\":1,\"productId\":126}],\"serial\":\"토키 시리얼 넘버\"}"
```

UI 상에서 입력받은 구매자 이름과 연락처를 넣어야 합니다.  
`buyerTel`에 입력된 번호로 결제 요청 문자가 발송됩니다.  
`approvalDTO`는 배열로, 구매할 상품이 한개일지라도 배열 안에 표기해야 합니다.  

사용자는 문자를 받은 후 결제 하고 다시 재생을 시도할 것입니다.  
재생을 시도하기 전에 `4. 유효성 조회` 항목의 요청을 다시 시도하십시오.

### 6. 결제 내역 조회
#### [GET] /api/toki/orders

사용자에게 결제 내역이 있을 경우 유통사나 콘텐츠 공급사의 정지와 무관하게 재생이 보장되어야 합니다.  
결제 내역에는 콘텐츠의 승인 정보가 포함되어 있으므로 앱 초기화 단계에서 분기 처리에 활용할 수 있습니다.

|파라미터명    |필수    |기본값|설명                  |
|------------|-------|-----|---------------------|
|serial      |**예** |     |시리얼 넘버            |
|page        |아니오  |0    |페이지네이션 위치       |

다음과 같이 호출할 수 있습니다.

```shell script
curl -X GET "https://h1appstore.hancom-toki.com:9443/api/toki/orders?serial={serialKey}" -H "accept: */*"
```

요청이 성공하면 다음과 같은 응답을 받을 수 있습니다.

```json
{
  "total": 5,
  "orders": [
    {
      "id": 123,
      "status": "paid",
      "cancelReason": null,
      "deliveryStart": null,
      "deliveryEnd": null,
      "paymentAt": "2020-12-03T05:02:22.000+00:00",
      "paymentType": "SMS카드결제",
      "paymentId": "20201203OD000009",
      "paymentName": "정글비트 그린",
      "pgReturnData": "{\"mid\":\"pghancom0m\",\"moid\":\"20201203OD000009\",\"goodsName\":\"정글비트 그린\",\"amt\":\"2000\",\"dutyFreeAmt\":null,\"buyerName\":\"토키 고객님\",\"buyerTel\":\"01058336062\",\"buyerEmail\":null,\"payExpDate\":null,\"userId\":null,\"resultCode\":\"0000\",\"resultMsg\":\"[한컴로보틱스]\\n상품명 : 정글비트그린\\n상품금액 : 2,000원\\nSMS 안심결제가 요청되었습니다\\n\\n안심결제 접속URL안내\\n<a href='https://innopay.kr/hVGolXZ74e' target='_blank'>https://innopay.kr/hVGolXZ74e</a>\\n\\n- 결제대행사 (주)인피니소프트 이노페이\",\"orderUrl\":\"https://innopay.kr/hVGolXZ74e\",\"orderContents\":\"[한컴로보틱스]\\n상품명 : 정글비트그린\\n상품금액 : 2,000원\\nSMS 안심결제가 요청되었습니다\\n\\n안심결제 접속URL안내\\nhttps://innopay.kr/hVGolXZ74e\\n\\n- 결제대행사 (주)인피니소프트 이노페이\"}",
      "amt": 2000,
      "pgTid": "pghancom0m01032012031154493144",
      "buyerEmail": null,
      "buyerName": "고객님",
      "buyerTel": "01058336062",
      "zoneCode": "02603",
      "address": "서울 동대문구 사가정로 6",
      "addressDetail": "1",
      "recipientName": "고객님",
      "recipientPhoneNo": "01058336062",
      "comment": "",
      "approvals": [
        {
          "id": 234,
          "created_at": "2020-12-03T02:54:37.000+00:00",
          "device_id": 10,
          "expired_at": "2021-12-03T02:54:37.000+00:00",
          "order_id": 123,
          "product_id": 345,
          "updated_at": "2020-12-03T02:54:37.000+00:00",
          "distributor_id": 20
        },
        "...반복..."
      ],
      "createdAt": "2020-12-03T02:54:37.000+00:00",
      "updatedAt": "2020-12-03T05:02:22.000+00:00"
    },
    "...반복..."
  ]
}
```

응답 데이터 중 approvals 배열에는 유효한 승인 정보가 표시됩니다.  
결제 취소를 통해 승인이 취소 되거나 승인이 만료되어 더 이상 유효하지 않게 되면 목록에서 사라집니다.  
따라서 가용 컨텐츠가 없을 경우에는 approvals 는 빈 배열이 됩니다.

끝.

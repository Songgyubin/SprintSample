# LGR 리포트 선택 검색 공통 API 전환 안내

작성일: 2026-08-10  
대상: 리포트 작성 화면의 근무지·유통사·브랜드 선택 검색 호출처

## 요약

LGR에 있던 선택 검색 API는 리포트/보고서함 데이터가 아닌 고객사 내 마스터 데이터를 조회합니다. 이에 따라 근무지·유통사는 workplace 도메인의 공통 API로 분리했고, 브랜드는 기존 FM 검색 API를 `brandIds` optional 필터로 확장해 사용합니다.

| 선택 대상 | 기존 LGR 경로 (admin / rest) | 공통 API 경로 (admin / rest) | 상태 |
| --- | --- | --- | --- |
| 근무지 | `POST /admin/lgr/v2/report/workplace/search`<br>`POST /rest/lgr/v2/report/workplace/search` | `POST /admin/workplace/v3/selection/search`<br>`POST /rest/workplace/v3/selection/search` | 구현 완료 (현재 브랜치, 배포 전) |
| 유통사 | `POST /admin/lgr/v2/report/retailer/search`<br>`POST /rest/lgr/v2/report/retailer/search` | `POST /admin/workplace/v3/selection/retailer/search`<br>`POST /rest/workplace/v3/selection/retailer/search` | 구현 완료 (현재 브랜치, 배포 전) |
| 브랜드 | `POST /admin/lgr/v2/report/brand/search`<br>`POST /rest/lgr/v2/report/brand/search` | `GET /admin/fm/v2/brand/search`<br>`GET /rest/fm/v2/brand/search` | `brandIds` optional 확장 구현 필요 |

> 기존 LGR 경로는 클라이언트 전환과 운영 호출 확인이 끝날 때까지 유지한다. 신규 경로는 로그인 사용자의 고객사 범위에서만 조회한다.

## 공통 페이징 응답

아래 목록 API는 `PageResponse` 형태를 사용한다.

```json
{
  "content": [],
  "page": 0,
  "size": 20,
  "total": 0,
  "totalElements": 0,
  "last": true
}
```

- `content`: 현재 페이지의 항목 목록
- `page`: 0부터 시작하는 현재 페이지 번호
- `size`: 요청 페이지 크기
- `total` / `totalElements`: 전체 항목 수
- `last`: 마지막 페이지 여부

## 근무지

### 기존 LGR API

#### `POST /admin/lgr/v2/report/workplace/search`

#### `POST /rest/lgr/v2/report/workplace/search`

- Request Body

```json
{
  "workplaceIds": ["workplace-1"],
  "searchText": "강남",
  "page": 0,
  "records": 20
}
```

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | :---: | --- |
| `workplaceIds` | string[] | 아니오 | 선택 가능한 근무지 ID 목록. 빈 배열이면 고객사의 전체 활성 근무지 조회 |
| `searchText` | string | 아니오 | 근무지명, 근무지 코드, 주소, 상세 주소 검색어 |
| `page` | number | 아니오 | 페이지 번호, 기본값 `0` |
| `records` | number | 아니오 | 페이지 크기, 기본값 `20` |

- Response Body

```json
{
  "content": [
    {
      "workplaceId": "workplace-1",
      "workplaceName": "강남점",
      "workplaceCode": "GN-01"
    }
  ],
  "page": 0,
  "size": 20,
  "total": 1,
  "totalElements": 1,
  "last": true
}
```

### 전환 대상 공통 API

#### `POST /admin/workplace/v3/selection/search`

#### `POST /rest/workplace/v3/selection/search`

- Request Body와 Response Body는 기존 LGR 근무지 검색 API와 **동일**하다.
- 조회 규칙도 동일하다.
  - 고객사 내 활성 근무지만 조회
  - `workplaceIds`가 있으면 해당 ID와 교집합만 조회
  - 이름 → ID 오름차순 정렬
- 오류: `page < 0` 또는 `records < 1`이면 `400 INVALID_PARAMS`
- 변경 사항 요약: 호출 경로만 LGR에서 workplace 도메인으로 변경한다.

## 유통사

### 기존 LGR API

#### `POST /admin/lgr/v2/report/retailer/search`

#### `POST /rest/lgr/v2/report/retailer/search`

- Request Body

```json
{
  "retailerIds": ["retailer-1"],
  "searchText": "Shopl",
  "page": 0,
  "records": 20
}
```

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | :---: | --- |
| `retailerIds` | string[] | 아니오 | 선택 가능한 유통사 ID 목록. 빈 배열이면 고객사의 전체 활성 유통사 조회 |
| `searchText` | string | 아니오 | 유통사명 검색어 |
| `page` | number | 아니오 | 페이지 번호, 기본값 `0` |
| `records` | number | 아니오 | 페이지 크기, 기본값 `20` |

- Response Body

```json
{
  "content": [
    {
      "retailerId": "retailer-1",
      "retailerName": "Shopl Retail",
      "retailerLogo": "https://example.com/retailer.png"
    }
  ],
  "page": 0,
  "size": 20,
  "total": 1,
  "totalElements": 1,
  "last": true
}
```

### 전환 대상 공통 API

#### `POST /admin/workplace/v3/selection/retailer/search`

#### `POST /rest/workplace/v3/selection/retailer/search`

- Request Body와 Response Body는 기존 LGR 유통사 검색 API와 **동일**하다.
- 조회 규칙도 동일하다.
  - 고객사 내 활성 유통사만 조회
  - `retailerIds`가 있으면 해당 ID와 교집합만 조회
  - 이름 → ID 오름차순 정렬
- 오류: `page < 0` 또는 `records < 1`이면 `400 INVALID_PARAMS`
- 변경 사항 요약: 호출 경로만 LGR에서 workplace 도메인으로 변경한다.

## 브랜드

### 기존 LGR API

#### `POST /admin/lgr/v2/report/brand/search`

#### `POST /rest/lgr/v2/report/brand/search`

- Request Body

```json
{
  "brandIds": ["brand-1"],
  "searchText": "Shopl",
  "page": 0,
  "records": 20
}
```

- Response Body

```json
{
  "content": [
    {
      "brandId": "brand-1",
      "brandName": "Shopl Brand",
      "brandLogo": "https://example.com/brand.png"
    }
  ],
  "page": 0,
  "size": 20,
  "total": 1,
  "totalElements": 1,
  "last": true
}
```

### 전환 대상 FM 공통 API

#### `GET /admin/fm/v2/brand/search`

#### `GET /rest/fm/v2/brand/search`

- Query Parameter

| 필드 | 타입 | 필수 | 설명 |
| --- | --- | :---: | --- |
| `brandIds` | string[] | 아니오 | **추가 예정.** 전달 시 해당 브랜드 ID와 교집합만 조회 |
| `searchText` | string | 아니오 | 브랜드명 검색어 |
| `categoryIdList` | string[] | 아니오 | 제품군 ID 필터 |
| `page` | number | 아니오 | 페이지 번호, 기본값 `0` |
| `records` | number | 아니오 | 페이지 크기, 기본값 `20` |

- Response Body

```json
{
  "content": [
    {
      "brandId": "brand-1",
      "brandName": "Shopl Brand",
      "brandLogo": "https://example.com/brand.png",
      "brandLogoThumbnail": "https://example.com/brand-thumbnail.png",
      "regDt": "2026-08-10T09:00:00"
    }
  ],
  "page": 0,
  "size": 20,
  "total": 1,
  "totalElements": 1,
  "last": true
}
```

- 변경 사항 요약
  - HTTP method가 `POST`에서 `GET`으로 변경된다.
  - `brandIds`는 optional query parameter로 추가한다. 미전달 시 기존 FM API 동작을 변경하지 않는다.
  - 응답에는 `brandLogoThumbnail`, `regDt`가 추가로 포함될 수 있다.
  - 정렬은 기존 FM API의 등록일시 → 브랜드명 오름차순을 유지한다. 기존 LGR의 브랜드 유형 우선 정렬은 적용하지 않는다.

## 클라이언트 전환 체크리스트

- [ ] 근무지 호출을 `/workplace/v3/selection/search`로 변경
- [ ] 유통사 호출을 `/workplace/v3/selection/retailer/search`로 변경
- [ ] 브랜드 `brandIds` 확장 배포 후 FM `GET /fm/v2/brand/search` 호출로 변경
- [ ] GET 전환 시 request body를 query parameter로 변환
- [ ] 브랜드 응답의 추가 필드를 허용하고, 기존 LGR 정렬 의존 여부 확인
- [ ] LGR 기존 경로 제거 전 운영 호출 및 앱 최소 버전 확인

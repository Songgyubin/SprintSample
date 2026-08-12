# 근무지 선택 검색 API

## `POST /rest/workplace/v3/selection/search`

### Request Body

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

### Response Body

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

### 조회 규칙

- 고객사 내 활성 근무지만 조회
- `workplaceIds`가 있으면 해당 ID와 교집합만 조회
- 이름 → ID 오름차순 정렬
- `page < 0` 또는 `records < 1`이면 `400 INVALID_PARAMS`

# Full-text index 사용

Full-text index는 검색 기능을 향상시키기 위해 사용하는 인덱스 기법이다.

기존에 앞 글자만 인덱스를 생성해주던 방식과 달리 중간 글자에도 인덱스를 걸어서 검색 시간을 단축시켜준다.

## 왜 쓰는가

일반적인 B-Tree 인덱스는 `LIKE '%검색어%'` 같은 형태는 인덱스를 지원하지 않아서 데이터를 전체 스캔해야하는 문제가 발생한다.

특히 긴 텍스트, 문자열을 검색해야 할 경우 검색 시간은 훨씬 오래 걸리므로 서버에 치명적인 문제를 발생시킬 수 있다.

Full-text index는 글자 단위로 인덱스를 생성해주기 때문에 전체 텍스트 검색 시간을 대폭 줄일 수 있다.

## 사용 방법

```sql
CREATE FULLTEXT INDEX 인덱스이름 ON 테이블이름 (열이름);
```

### 자연어 검색

```sql
SELECT * FROM 테이블 WHERE MATCH(컬럼) AGAINST('키워드');
```

- 정확한 단어 검색('키워드'가 포함된 것)
- 키워드를, 키워드가 등 조사가 붙어있으면 검색 안됨

### boolean 모드 검색

> 단어나 문장이 정확히 일치하지 않는 것도 검색

```sql
-- '키워드' 를 찾되 반드시 '필수'라는 글자가 들어가있는 컬럼
SELECT * FROM 테이블 WHERE MATCH(컬럼) AGAINST('키워드 +필수' IN BOOLEAN MODE);
```

| 연산자 | 내용 | 예시 |
| --- | --- | --- |
| + | 검색 필수 |
| - | 제외 |
| ~ | 부정, 우선순위가 낮음 ('키워드' 를 찾되 '필수'라는 글자가 없는 컬럼보다 아래 순위) | `SELECT * FROM newspaperWHERE MATCH(article) AGAINST('영화 ~액션' IN BOOLEAN MODE);` |
| * | 부분 검색, 조사 등 |
| " | 정확한 구문, “재밌는 영화”, “재밌는 영화가” 등 | `SELECT * FROM newspaperWHERE MATCH(article) AGAINST("재밌는 영화" IN BOOLEAN MODE);` |


# 3장 인덱스 튜닝
## 3.1 테이블 액세스 최소화
> SQL 튜닝 = 랜덤 I/O 최소화 = 테이블 랜덤 액세스 최소화
#### ROWID를 이용한 테이블 액세스
- ROWID를 이용한 테이블 액세스는 생각보다 고비용을 구조다.
	- 모든 데이터가 캐싱되어 있더라도 테이블 레코드를 찾기 위해 매번 DBA 해싱과 래치 획득 과정을 반복해야 하고, 동시 액세스가 심할 때는 캐시 버퍼 체인 래치와 버퍼 Lock에 대한 경합까지 발생하기 때문이다.
- 오라클에서 하나의 레코드를 찾아가는데 있어 가장 빠르다고 알려진 'ROWID에 의한 테이블 액세스'가 얼마나 고 비용연산인지 이해하길 바란다.
#### 인덱스 클러스터링 팩터
- 클러스터링 팩터(CF)
	- 군집성 계수로 번역
	- 특정 컬럼을 기준으로 같은 값을 갖는 데이터가 서로 모여있는 정도
- CF가 좋은 컬럼에 생성한 인덱스는 검색 효율이 매우 좋다
- CF가 안 좋은 인덱스를 사용하면 테이블을 액세스하는 횟수 만큼 고스란히 블록 I/O가 발생한다
#### 인덱스 손익분기점
- 인덱스 손익분기점
	- Index Range Scan에 의한 테이블 액세스가 Table Full Scan보다 느려지는 지점
- Index Range Scan이 Table Full Scan보다 느려지게 만드는 요인
	- Table Full Scan은 시퀀셜 액세스인 반면, 인덱스 ROWID를 이용한 테이블 액세스는 랜덤 액세스 방식이다
	- Table Full Scan은 Multiblock I/O인 반면, 인덱스 ROWID를 이용한 테이블 액세스는 Single Block I/O 방식이다
#### 온라인 프로그램 튜닝 vs 배치 프로그램 튜닝
- 온라인 프로그램 튜닝
	- 온라인 프로그램은 보통 소량의 데이터를 읽고 갱신하므로 인덱스를 효과적으로 사용하는 것이 중요
- 배치 프로그램 튜닝
	- 배치 프로그램은 대량 데이터를 읽고 갱신해야하므로 항상 전체 범위 처리 기준으로 튜닝
	- 대량 데이터를 빠르게 처리하려면 인덱스와 NL 조인보다 Full Scan과 해시 조인이 유리
	- 초대용량 테이블을 Full Scan하면 비용이 크므로 파티션을 활용하는 것이 매우 중요한 튜닝 요소
#### 테이블 액세스 최소화를 위한 튜닝 기법
- 인덱스 컬럼 추가
#### IOT(Index-Organized Table)
- 랜덤 엑세스가 발생하지 않도록 생성한 인덱스 구조
- 테이블 블록에 있어야할 데이터를 인덱스 리프 블록에 저장
#### 클러스터 테이블
- 클러스터 테이블 종류
	- 인덱스 클러스터 테이블
		- 클러스터 키 값이 같은 레코드를 한 블록에 모아서 저장하는 구조
	- 해시 클러스터 테이블
		- 인덱스 대신 해시 알고리즘을 사용해 클러스터를 찾아가는 구조
---
## 3.2 부분범위 처리 활용
#### 부분 범위 처리
- 사용자로부터 Fetch Call이 있을 때마다 일정량씩 나누어 전송하는 것
- 데이터를 전송하는 단위인 Array Size는 클라이언트 프로그램에서 설정
#### 정렬조건이 있는 경우
- 정렬하는 컬럼이 인덱스인 경우 부분범위 처리 가능
- 정렬하는 컬럼이 인덱스가 아닌 경우 부분 범위 처리 불가능
#### 적정 Array Size
- 무조건 모든 데이터가 필요한 경우 Array Size를 가능한 크게
- 앞쪽 일부 데이터만 필요한 경우 Array Size를 작게 설정하는 것이 유리
---
## 3.3 인덱스 스캔 효율화
#### 액세스 조건과 필터 조건
- 인덱스 액세스 조건
	- 인덱스 스캔 범위를 결정하는 조건절
- 인덱스 필터 조건
	- 테이블로 액세스할 지 결정하는 조건절
- 테이블 필터 조건
	- 쿼리 수행 다음 단계로 전달하거나 최종 결과집합에 포함할지 결정하는 조건절
#### 인덱스를 이용한 테이블 액세스 비용
- 비용 = 인덱스 수직적 탐색 비용 + 인덱스 수평적 탐색 비용 + 테이블 랜덤 액세스 비용
	  = 인덱스 루트와 브랜치 레벨에서 읽는 블록수 + 인덱스 리프 블록을 스캔하는 과정에서 읽는 블록수 + 테이블 액세스 과정에서 읽는 블록수
#### 비교 연산자 종류와 컬럼 순서에 따른 군집성
- 첫 번째 나타나는 범위 검색 조건까지만 만족하는 인덱스 레코드는 모두 연속해서 모여있지만, 그 이하 조건까지 만족하는 레코드는 비교연산자 종류에 상관 없이 흩어진다(우연히 모여있을 수는 있음)
- 선행 컬럼이 모두 '=' 조건인 상태에서 첫 번째 나타나는 범위 검색 조건이 인덱스 스캔 범위를 결정한다
- 첫번째 나타나는범위 검색 조건까지가 인덱스 액세스 조건이고, 나머지는 필터조건이다.
#### 인덱스 선행 컬럼이 등치(=) 조건이 아닐 때 생기는 비효율
- 인덱스 스캔 효율성은 인덱스 컬럼을 모두 등치(=) 조건으로 사용할 때 가장 좋다
	- 조건을 만족하는 레코드가 모두 한데 모여있기 때문이다.
- 인덱스 컬럼 중 일부가 조건절에 없거나 등치 조건이 아니더라도, 그것이 뒤쪽 컬럼일 때는 비효율이 없다.
- 인덱스 선행 컬럼이 조건절에 없거나 부등호, BETWEEN, LIKE같은 범위검색 조건이면, 인덱스를 스캔하는 단계에서 비효율이 생긴다.
#### BETWEEN을 IN-LIST로 전환
- In-List 개수 만큼 UNION ALL 브랜치가 생성되고 각 브랜치마다 모든 컬럼은 '=' 조건으로 검색하므로 앞서 선두 컬럼에 BETWEEN을 사용할 때와 같은 비효율이 사라진다
- 단, In-List의 개수가 많은 경우 수직적 탐색이 많이 발생하므로 BETWEEN 조건 때문이 리프 블록을 많이 스캔하는 비효율보다 In-List 개수 만큼 브랜치 블록을 반복 탐색하는 비효율이 더 클 수 있다
- 인덱스 스캔 과정에서 선택되는 레코드들이 서로 멀리 떨어져있을 때에만 유용하다
#### Index Skip Scan 활용
- 선두 컬럼이 BETWEEN이어서 나머지 검색 조건을 만족하는 데이터들이 서로 멀리 떨어져 있을 때, Index Skip Scan이 유용하다
#### PL/SQL 사용자 정의 함수가 느린 이유
- VM 상에서 실행되는 인터프리터 언어
- 호출 시마다 컨텍스트 스위칭 발생
- 내장 SQL에 대한 Recursive Call 발생
---
## 3.4 인덱스 설계
#### 인덱스가 많으면 생기는 문제
- DML 성능 저하
- 데이터베이스 사이즈 증가
- 데이터베이스 관리 및 운영 비용 증가
#### 인덱스 선택 기준
- 조건절에 항상 사용하거나 자주 사용하는 컬럼
- '=' 조건으로 자주 조회하는 컬럼을 선행 컬럼을 앞쪽으로
- 그외
	- 수행빈도, 업무상 중요도, 클러스터링 팩터, 데이터량, DML 부하, 저장공간, 인덱스 관리 비용

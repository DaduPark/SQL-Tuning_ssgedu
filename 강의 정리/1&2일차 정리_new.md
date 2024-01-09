## 데이터베이스 저장구조
<img width="300" alt="image" src="https://github.com/DaduPark/ReadingRecord/assets/76692927/f6dd4eb5-070d-4237-bf36-60f706ee45f3">  

<img width="486" alt="image" src="https://github.com/DaduPark/ReadingRecord/assets/76692927/0dd53276-4f7d-43ab-bc4d-8e5532325264">  

- 블록 
  - DBMS가 데이터를 읽고 쓰는 단위
  - 한 블록에 저장된 레크드는 모두 같은 테이블 레코드
  - 한 익스텐트에 담긴 블록은 모두 같은 테이블 블록
  - 오라클은 기본 8KB 
  - 테이블 뿐 아니라 인덱스도 블록단위로 데이터를 읽고 쓴다.
  - **데이터베이스 오브젝트중 가장 작은 단위**
  ```sql
  SHOW PARAMETER BLOCK_SIZE;
  ----------------------------------------------------
  db_block_size integer 8192   
  ```
- 익스텐트
  - 공간을 확장하는 단위. 연속된 블록 집합
  - 서로 다른 데이터 파일에 위치할 가능성이 높다(파일 경합을 줄이기 위해 여러 데이터파일로 분산 저장)
  - 인스텐트 내 블록은 서로 인접한 연속된 공간 / 익스텐트끼리는 연속된 공간이 아니다
- 세그먼트 
  - 데이터 저장공간이 필요한 오브젝트(테이블, 인덱스, 파티션, LOB 등)
  - 테이블이나 인덱스가 파티션 구조일 시 각 파티션이 하나의 세그먼트
- 테이블스페이스 
  - 세그먼트를 담는 콘테이너
  - 여러개의 데이터파일(DBF파일)로 구성
- 데이터 파일 : 디스크상의 물리적인 OS 파일

#### 성능을 위해서는 시간과 읽기 수를 줄여야한다
> 데이터 조작작업(insert, update, delete) 후 가끔 압축을 해줘야 성능이 좋아짐.  
> 예로 update 시 해당 데이터가 존재하는 페이지에만 간단히 데이터를 수정되는 것이 아니라 페이지 스플릿, 페이지 생성등 작업이 진행되며 페이지가 늘어가게 되어 I/O가 많아 지게 되므로 성능이 저하된다. 이로 인해 압축작업을 진행해줘야한다.

#### [참고] 오라클의 논리적 구성요소
데이터 블록(data blocks) -> 확장 영역(extents) -> 세그먼트(segments) -> 테이블스페이스(tablespace) -> 데이터베이스(database)

#### [참고] 오라클의 물리적 구성요소 : DBF(Data File)
- 오라클 구성정보 파일 위치 => D:\app\\(유저명)\oradata\orcl 
- REDOXX.log 파일 => DB에서 물리적으로 데이터가 변하는것을 기록 (DML_insert, update, delete)  
- XX.DBF 파일 (테이블스페이스명) => 테이블 스페이스에서 사용하는 데이터 파일
- USERSXX.DBF 파일 (사용자스페이스명) => 사용자 계정이 사용하는 데이터 파일
![image](https://github.com/DaduPark/SQL-Tuning_ssgedu/assets/134204844/b511a7f0-5557-4fb1-b22d-aa813dbd9426)

## ROWID
- 테이블 레코드를 가리키는 주소값
- 데이터블록과 로우번호로 구성되어있으므로 이값을 알면 데이터 레코드를 찾아갈 수 있다.
> ROWID = 테이블 블록 주소 + 로우번호  
> 테이블 블록 주소 = 데이터 파일 번호 + 블록 번호
> 블록 번호 : 데이터파일 내에서 부여한 상대적 순번
> 로우번호 : 블록 내 순번

### ROWID 실습
```sql
select rowid, empno, ename from emp;
----------------------------------------------------
AAAR3sAAEAAAACXAAA	7369	SMITH
AAAR3sAAEAAAACXAAB	7499	ALLEN
AAAR3sAAEAAAACXAAC	7521	WARD
AAAR3sAAEAAAACXAAD	7566	JONES
AAAR3sAAEAAAACXAAE	7654	MARTIN
AAAR3sAAEAAAACXAAF	7698	BLAKE
AAAR3sAAEAAAACXAAG	7782	CLARK
AAAR3sAAEAAAACXAAH	7788	SCOTT
AAAR3sAAEAAAACXAAI	7839	KING
AAAR3sAAEAAAACXAAJ	7844	TURNER
AAAR3sAAEAAAACXAAK	7876	ADAMS
AAAR3sAAEAAAACXAAL	7900	JAMES
AAAR3sAAEAAAACXAAM	7902	FORD
AAAR3sAAEAAAACXAAN	7934	MILLER
```

- SMITH의 rowid : AAAR3sAAEAAAACXAAA 
  - AAAR3s : Object번호 
  - AAE : File번호
  - AAAACX : Block 번호
  - AAA : 위치번호(페이지내의)
  => 한페이지에 있으므로 위치번호를 제외하고 다 같다

```sql
select rowid, empno, ename from emp  
union   
select rowid, deptno, dname from dept;  
----------------------------------------------------
AAAR3qAAEAAAACHAAA	10	ACCOUNTING
AAAR3qAAEAAAACHAAB	20	RESEARCH
AAAR3qAAEAAAACHAAC	30	SALES
AAAR3qAAEAAAACHAAD	40	OPERATIONS
AAAR3sAAEAAAACXAAA	7369	SMITH
AAAR3sAAEAAAACXAAB	7499	ALLEN
AAAR3sAAEAAAACXAAC	7521	WARD
AAAR3sAAEAAAACXAAD	7566	JONES
AAAR3sAAEAAAACXAAE	7654	MARTIN
AAAR3sAAEAAAACXAAF	7698	BLAKE
AAAR3sAAEAAAACXAAG	7782	CLARK
AAAR3sAAEAAAACXAAH	7788	SCOTT
AAAR3sAAEAAAACXAAI	7839	KING
AAAR3sAAEAAAACXAAJ	7844	TURNER
AAAR3sAAEAAAACXAAK	7876	ADAMS
AAAR3sAAEAAAACXAAL	7900	JAMES
AAAR3sAAEAAAACXAAM	7902	FORD
AAAR3sAAEAAAACXAAN	7934	MILLER
```
- 파일은 같다 : scott과 hr은 일반계정으로 데이터스페이스가 같으므로 저장되는 파일이 같고 다른 테이블이므로 OBject가 다르며 각 테이블 마다 동일한 페이지에 존재하므로 Block번호가 같으며 다른 ROW들이므로 위치번호가 모두 다르다

##  I/O 메커니즘
### 시퀀셜 액세스 VS 랜덤 액세스
- 시퀀셜 액세스
  -  논리적, 물리적으로 연결된 순서에 따라 차례로 읽고 쓰는 방식
  -  인덱스 리프 블록은 앞뒤를 가리키는 주소값을 통해 논리적으로 서로 연결되어있음
  -  Full Table Scan시 사용
- 랜덤 엑세스
  - 레코드 하나를 읽기 위해 한 블록씩 접근하는 방식
  - 인덱스에서 테이블 엑세스 하는것도 랜덤 엑세스
    
### 논리적 I/O VS 물리적 I/O
- 서버프로세스와 데이터파일 사이에 **DB 버퍼 캐시**가 있어 데이터파일을 통해 한번 읽은 데이터를 저장한다.(버퍼캐시는 공유메모리 영역이므로 같은 블록을 읽는 다른 프로세스도 득을 본다)
- DB버퍼캐시는 **데이터 캐시**라고 할 수 있다.(라이브러리 캐시는 코드 캐시라고 할 수 있다.)
- 논리적 I/O
   - SQL문 처리과정에서 메모리 버퍼 캐시에서 발생한 총 블록 I/O
- 물리적 I/O
   - 버퍼캐시에서 존재하지 않아 디스크를 읽은 총 I/O
   - 상당히 느림
   - 물리적 I/O = 논리적 I/O x (100 - BCHR)
   - BCHR : 버퍼캐시히트율
- 물리적 I/O는 시스템 상황에 의해 결정되는 통제 불가능한 외생변수
- SQL성능을 높이기 위해서 할 수 있는 일은 논리적 I/O를 줄이는 일 뿐이다 (I/O자체를 줄이라는 뜻)

### Single Block I/O VS Multiblock I/O
- Single Block I/O
  - 한 블록씩 요청해서 메모리에 적재하는 방식
  - 인덱스를 이용할때 기본적으로 사용
- Multiblock I/O
  - 한번에 여러 블록씩 요청해서 메모리에 적재하는 방식
  - 테이블 전체 스캔할때 사용
  - 여러 블록을 요청함으로써 프로세스의 대기 큐(wait queue)를 줄여 성능을 높일 수 있다
  - 여러 인스텐트에 존재하는 값을 가져오지못한다. 즉 한 인스텐트의 값을 가져옴
 
### Table Full Scan VS Index Range Scan
- Table Full Scan
   - 테이블 전체 스캔
   - 시퀀셜 액세스와 Multiblock I/O 사용
- Index Range Scan
   - 인덱스를 이용해서 읽는 방식
   - 랜덤 엑세스와 Single Block I/O 사용

- 인덱스 사용은 중요하나 읽을 데이터의 일정량이 넘으면 인덱스보다 Table Full Scan이 더 유리하다
## 인덱스
### Heap 및 B트리구조 인덱스의 이해
#### Heap
- 데이터가 무작위로 한 줄로 나열하여 페이지에 담긴다.
- 데이터가 무작위이므로 조회성능은 떨어짐
- 데이터 조작(insert, update, delete)을 할 때는 데이터를 무작위로 넣어도 되므로 조작 성능은 좋다.
- 조회 : 데이터페이지 스캔
- Full Scan

#### B트리구조 인덱스
- 데이터가 인덱스컬럼 기준으로 트리형태로 페이지에 담긴다.
- 루트및 브랜치에 저장된 각 인덱스 레코드는 하위 블로에 대한 주소값을 갖는다(수직적 탐색이 가능)
- 데이터가 트리형태로 나열되어있으므로 인덱스 컬럼 조건에 맞는 조회를 진행 시 조회성능이 좋다.
- 데이터 조작(insert, update, delete)을 할 때는 리브, 브랜치, 루트 레벨의 전체 페이지에 영향이 갈 수 있으므로 조작성능이 떨어짐(불필요한 인덱스 생성은 줄여야함)
- 인덱스 구조를 한줄로 할 경우 조작할 때 전체 페이지의 수정이 작업되야하므로 db가 다운된다 -> B-tree구조 탄생 (트리구조)
- 조회 : 인덱스페이지 스캔 이후 해당 데이터의 ROWID(페이지주소+페이지 내의 데이터 순번)을 찾아 데이터페이지에서 스캔한다.
- Index Scan

### 인덱스가 존재하는 제약조건
- 제약조건(6개) : PK, UK, FK, Not NULL, CHECK제약, (DEFAULT 제약)
- 인덱스가 걸리는 제약 조건 : PK, UK

### 인덱스 탐색 과정
   - 수직적 탐색
      - root-> leaf로 스캔
      -  인덱스 스캔 시작점을 찾는과정
      -  루트, 브랜치은 각 인덱스 레코드의 하위 블록에 대한 주소값을 갖기 때문에 수직적 탐색이 가능
      -  조건을 만족하는 첮번째 레코드를 찾는 과정
   - 수평적 탐색
     - 데이터를 찾는 과정(리프 블록을 수평적으로 수평)
     - 조건절을 만족하는 데이터를 모두 찾고 ROWID를 얻기 위해 탐색
     - 같은 레벨(leaf레벨, branch레벨)에서 다음페이지로 스캔, leaf에서 다음것들 스캔(인덱스스캔 후 RowId를 통해 테이터 스캔 진행)

### 옵티마이저
- 규칙, 비용을 따져서 최적의 조회 성능을 낼 수 있게 도와준다.
   - RBO :  규칙 기반 옵티마이저
   - CBO :  비용 기반 옵티마이저
- 인덱스가 적용된 컬럼이 조건절에 걸렸다고 해도 Index Scan이 Full Scan보다 많아질 경우 옵티마이저가 판단하여 Full Scan(데이터페이지로만 스캔)을 한다.

### 엑세스 성능 향상
- 랜덤엑세스 : 시퀀셜엑세스가 아닌 모든 것 (인덱스 랜덤엑세스, 테이블 랜덤엑세스)
   - 인덱스 랜덤엑세스 : 인덱스 스캔(루트, 브랜치, 리프)으로 랜덤엑세스 되는 것
   - 테이블 랜덤엑세스 : 인덱스 스캔 후 데이터스캔으로 랜덤엑세스 되는 것
- 시퀀셜엑세스 : 순서에 맞춰 찾아가는 것(다음페이지 이전페이지 찾아가는것)
- 성능을 향상시키기 위해서는 랜덤엑세스를 줄여라. (인덱스 랜덤엑세스가 아닌 테이블 랜덤엑세스를 줄이는 것이 성능에 좋다!!)

---------
## 인덱스 실습
> < 가정 >   
> - ROWID : 페이지와 위치만 생각한다.  
> - 인덱스에는 키값과 ROWID만 들어간다.
> - 데이터가 한 파일에 4개가들어간다.
> - 인덱스는 한 파일에 5개만 들어간다고 생각하고 본다.
> - T1테이블(no, name, ...) 한 로우(데이터)당 2000bytes.

#### 1. heap과 B-트리 구조 인덱스 데이터 저장

<img src="https://github.com/DaduPark/ReadingRecord/assets/76692927/4fcd5048-b1b4-4f50-b343-47da8be3b5c6" height="200"/>

>  => 17개행을 넣는다 > 데이터페이지 5페이지가 필요(4페이지에 4개씩 + 1페이지에 데이터 1개)
>  => 값이 랜덤으로 들어가있을 때 (힙) > table full Scan(값을 찾을때 5페이지를 스캔해야함) 

<img src="https://github.com/DaduPark/ReadingRecord/assets/76692927/01de69f0-b228-44fc-8de4-c742290ad8ff" height="200"/>

>  => 17행을 넣으므로 인덱스페이지(인덱스 기준 : name )는 Leaf 4개(3페이지에 5개씩 + 1페이지에 데이터 1개) + Branch, Root 7개 총 11페이지 필요
> 인덱스를 4페이지(17개)에 name순서에 맞춰서 쓰며 rowId(파일명, 순번)으로 나타냄-> branch 2개의 페이지에 하단의 상의 name을 가지고 만듦 > root 1페이지에 branch 상위 name을 가지고 만듦

#### 2. Search (1. heap과 B-트리 구조 인덱스 데이터 저장 사진 참고)
##### 2.1. Full scan
-  no을 조건절에 건다면 인덱스(name)를 타지 않기 때문에 데이터페이지 5페이지를 찾게된다.

##### 2.2. Index scan
-  name을 조건절에 건다면 인덱스를 타기 때문에 인덱스페이지 3페이지 + 데이터페이지 1페이지, 총 4페이지만 찾게된다.

##### 2.3. 옵티마이저에 의한 Full scan
- (where name between '디비' and '종이')의 조건절을 건다면 12개(Root-branch-leaf 3페이지 + 수평적탐색 + 데이터페이지스캔)
==> 옵티마이저가 인덱스 스캔이 많아질 경우에는 인덱스를 사용 안할 수 있다. 데이터페이지로만 확인함 (full 스캔)


#### 3. Insert('배달'데이터 추가)
##### 3.1. 데이터페이지 수정
<img src="https://github.com/DaduPark/ReadingRecord/assets/76692927/70df7f43-ddd7-4178-b28e-900f5f2e9a88" height="200"/>  

- 데이터 추가 시 데이터페이지는 마지막에 추가하면 됨

##### 3.2. 인덱스 페이지 수정
<img src="https://github.com/DaduPark/ReadingRecord/assets/76692927/a73a0ce2-6377-40a9-a113-8c6d7bdb94c0" height="200"/>  

- 인덱스페이지에는 leaf 레벨에 한장을 더 추가하여(페이지 스플릿_반띵) leaf(추가된 페이지과 기존 페이지를 순서에 맞게 주소값도 변경), branch등 많은 페이지들을 조정한다(페이지 추가 : 205, 페이지 수정 :202, 201)
( 조회에서는 인덱스가 빠르지만 조작관점으로 보면 heap이 성능이 좋다. 그러므로 불필요한 인덱스 생성은 줄여야한다.)


#### 4. Update ('모자'데이터를 '강자'데이터로 변경)
##### 4.1. 데이터페이지 수정
<img src="https://github.com/DaduPark/ReadingRecord/assets/76692927/e21d318d-8b12-4ffe-ae84-5dbdbf026603" height="200"/>  

- 해당 데이터만 수정되면됨
  
##### 4.2. 인덱스 페이지 수정
<img src="https://github.com/DaduPark/ReadingRecord/assets/76692927/a6663dfb-fd7d-49b0-ac12-7f00b72474d3" height="200"/>  

- 201페이지는 페이지 스플릿처리되어 반으로 나뉘게 되고 순서에 맞게 '강자'데이터는 201로 감
- 나머지 반은 206으로 변경
- branch레벨인 301에 206의 가장 상위 데이터가 셋팅됨
- 202에 '모자'데이터 삭제

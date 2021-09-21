# keeyong-data-engineering-batch5
# ->
## airflow study

#### 트렌젝션 구현하기
NameGenderCSVtoRedshift.py
PostgreSQL/psycopg2의 트랜잭션
autocommit이라는 파라미터로 조절가능
autocommit=True가 되면 기본적으로 PostgreSQL의 커밋 모드 사용
기본적으로 모든 SQL statement가 바로 커밋됨 (autocommit = True)
이를 바꾸고 싶다면 BEGIN;END; 혹은 BEGIN;COMMIT을 사용 (혹은 ROLLBACK)
autocommit=False가 되면 커넥션 객체의 .commit()과 .rollback()함수로 트랜잭션 조절 가능


```
-- 아래 같이 try/catch 사용해도 가능
-- autocommit=True로 실행하고 psycopg2로 컨트롤하기
try:
  cur.execute("DELETE FROM keeyong.name_gender;") 
  cur.execute("INSERT INTO keeyong.name_gender VALUES ('Claire', 'Female');")
  conn.commit()
except (Exception, psycopg2.DatabaseError) as error:
  print(error)
  conn.rollback()
finally :
  conn.close()
```

#### v2 개선
NameGenderCSVtoRedshift_v2.py

DAG Parameters 사용
max_active_runs: # of DAGs instance
concurrency: # of tasks that can run in parallel
catchup: whether to backfill past runs

DAG parameters vs. Task parameters의 차이점 이해가 중요
위의 파라미터들은 모두 DAG 파라미터로 DAG 객체를 만들 때 지정해주어야함
default_args로 지정해주면 에러는 안 나지만 적용이 안됨
default_args로 지정되는 파라미터들은 태스크 레벨로 적용되는 파라미터들


#### v3 개선
NameGenderCSVtoRedshift_v3.py

Xcom 객체를 사용해서 세 개의 task로 나누기
Redshift의 스키마와 테이블 이름을 params로 넘기기
Variable를 이용해 CSV parameter 넘기기


#### v4 개선
NameGenderCSVtoRedshift_v4.py

Redshift Connection 사용하기
Connection 사용하면 AutoCommit  False 로 설정됨!


#### 커맨드라인에서 task 실행
~$ airflor tasks list
~$ airflow tasks test my_first_dag print_hello 2021-09-01
test로 실행시 airflow DB에 저장되지 않음



#### 날씨 api 이용
Weather_to_Redshift.py, v2.py

Open Weathermap api
앞으로 7일의 날씨 정보 저장

Incremental Update 방식으로 수정


#### MySQL to Redshift
MySQL_to_Redshift.py, v2.py, v3.py

MySQL DAG를 execution_date을 사용하게 변경
지금은 항상 모두 변경하게 되어 있지만 이를 execution_date을 사용해서 해당 하는 날만 읽어오게 변경
이를 이용해서 backfill을 command line에서 실행해보기


#### 


#### Backfill 커맨드라인에서 실행
airflow dags backfill ­-s 2018-­07-­01 ­-e 2018-­07-­31 dag_id
(catchup True 세팅해야함)
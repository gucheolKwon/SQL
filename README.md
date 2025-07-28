# SQL
현재 회사에서 사용하는 것 postgreSQL, MySQL
python에서 sqlalchemy를 사용한다면, psycopg2 (postgreSQL), pymysql(MySQL) 써야한다.

```python
import pandas as pd
from sqlalchemy import create_engine, text

psql_engine = "postgrsql+psycopg2://user:password@host:port/dbname"
mysql_engine = "mysql+pymysql://user:passwor@host:port/dbname"

engine = create_engine(psql_engine, pool_size=10,
                        max_overflow=20, pool_recycle=1800)
# 심화내용 1. 커넥션 풀링 사용 : 대규모 트래픽이나 배치 작업에 활용 가능
# poolsize : 최대 커넥션 수, max_overflow : pool_size를 초과해 생성할 수 있는 커넥션 수, pool_recycle : 커넥션 재사용 시간(초)

query = "SELECT * FROM db_table"
df = pd.read_sql(query, engine)

# 심화내용 2. 트랜잭션 처리 : 복수의 쿼리작업을 하나의 트랜잭션으로 안전하게 처리하고자 할 때,
from sqlalchemy.orm import Session
with Session(engine) as session:
  # 여러 쿼리를 실행
  session.execute("INSERT ...")
  session.execute("UPDATE ...")
  session.commit() # 모두 성공할때만 반영

# 심화내용 3. 파라미터 바인딩 : 직접 문자열 포매팅 대신에 파라미터 바인딩을 사용하면 SQL injection 위험을 줄이고, 가독성이 좋아짐.
query = "SELECT * FROM db_table WHERE value :> threshold"
df = pd.read_sql(query, engine, params={"threshold: 100"})

# 심화내용 4. ORM (Object Relational Mapping) 활용 : 파이썬에서 객체와 클래스로 다룸
from sqlalchemy.orm import declarative_base, sessionmaker
from sqlalchemy import Column, Integer, String

Base = declarative_base()
class User(base):
  __tablename__ = "users"
  id = Column(Integer, Primary_key=True)
  name = Column(String)
  age = Column(Integer)

Session = sessionmaker()
session = Session()
# ORM 쿼리로 데이터 조회 후 DataFrame 저장
users = session.query(User).filter(User.age > 20).all()
df = pd.DataFrame([u.__dict__ for u in users]).drop("_sa_instance_state", axis=1, errors='ignore')

# 심화내용 5. 대량데이터 처리 (Chunking / Streaming) : 쿼리 결과가 매우 클 때는 한번에 DataFrame으로 읽지 않고, chunk 단위로 처리해 메모리 사용 효율을 올릴 수 있음
for chunk in pd.read_sql("SELECT * FROM latge_dbtable", engine, chunksize=10000):
  process(chunk)
  # chunk = DataFrame이며, 필요한 후처리 작업 수행

# 심화내용 6. Raw SQL과 ORM의 혼합
from sqlalchemy import text

sql = text("""
    SELECT u.id, u.name, COUNT(p.id) AS post_count
    FROM users u
    LEFT JOIN posts p ON u.id = p.user_id
    GROUP BY u.id, u.name
""")
df = pd.read_sql(sql, engine)

```

DB 정보는 .env를 사용한다.
```text
DB_USER=myuser
DB_PASS=mypassword
DB_HOST=localhost
DB_PORT=5432
DB_NAME=mydatabase
```
pip install python-dotenv 필요
```python
from dotenv import load_dotenv
import os

load_dotenv() # os.getenv()도 가능 
db_user = os.getenv("DB_USER")
db_pass = os.getenv("DB_PASS")
db_host = os.getenv("DB_HOST")
db_port = os.getenv("DB_PORT")
db_name = os.getenv("DB_NAME")

db_url = f"postgresql+psycopg2://{db_user}:{db_pass}@{db_host}:{db_port}/{db_name}"

```

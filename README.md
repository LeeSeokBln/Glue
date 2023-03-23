Glue ETL을 사용하려면 데이터가 저장 될 데이터베이스 테이블이 필요하기 때문에, job을 생성하기 전 크롤러와 데이터베이스를 생성한후 크롤러를 실행 시켜 테이블을 생성하여야 한다.
# ETL로 데이터 변환 예제 코드
이름을 대문자로 바꾸고 parquet형식으로 저장하는 코드
```
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

## @params: [JOB_NAME]
args = getResolvedOptions(sys.argv, ['JOB_NAME'])

sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# 데이터 소스 지정 (Glue Data Catalog에서 테이블)
datasource0 = glueContext.create_dynamic_frame.from_catalog(database="<DB_NAME>", table_name="<TABLE_NAME>", transformation_ctx="datasource0")

# 데이터 변환 예제: 이름을 대문자로 변경
def upper_case_name(dynamic_record):
    dynamic_record["name"] = dynamic_record["name"].upper()
    return dynamic_record

# Mapping을 사용하여 이름 변환 함수 적용
mapped_data = Map.apply(frame=datasource0, f=upper_case_name, transformation_ctx="mapped_data")

# 결과 데이터를 Parquet 형식으로 S3에 저장
output_path = "OUTPUT_S3_PATH/"
datasink0 = glueContext.write_dynamic_frame.from_options(frame=mapped_data, connection_type="s3", connection_options={"path": output_path}, format="parquet", transformation_ctx="datasink0")

job.commit()

```

# ETL로 데이터 변환하기 예시코드
```
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job
from awsglue.dynamicframe import DynamicFrame

args = getResolvedOptions(sys.argv, ["JOB_NAME"])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args["JOB_NAME"], args)

# Script generated for node S3 bucket
S3bucket_node1 = glueContext.create_dynamic_frame.from_options(
    format_options={"multiline": False},
    connection_type="s3",
    format="json",
    connection_options={"paths": ["s3경로"], "recurse": True},
    transformation_ctx="S3bucket_node1",
)


datasource = glueContext.create_dynamic_frame.from_catalog(database = "데이터베이스 이름", table_name = "테이블 이름")
# Script generated for node ApplyMapping
applymapping = ApplyMapping.apply(frame = datasource,
                                 mappings = [("name", "string", "name", "string"),
                                             ("age", "int", "age", "int")],
                                 transformation_ctx = "applymapping")

# ApplyMapping 변환 결과 데이터 프레임
ApplyMapping2_node = applymapping


# Script generated for node S3 bucket
S3bucket_node3 = glueContext.write_dynamic_frame.from_options(
    frame=ApplyMapping2_node,
    connection_type="s3",
    format="json",
    connection_options={"path": "s3경로", "partitionKeys": []},
    transformation_ctx="S3bucket_node3",
)

job.commit()

```
Glue ETL을 사용하려면 데이터가 저장 될 데이터베이스 테이블이 필요하기 때문에, job을 생성하기 전 크롤러와 데이터베이스를 생성한후 크롤러를 실행 시켜 테이블을 생성하여야 한다.

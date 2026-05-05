# Test datasets for Apache Hive

A repository holding datasets and dumps that are used for testing Apache Hive.
The data can be found in the GitHub release assets. 
They are not under version control since they are in general large and not expected to change very often.
In addition, some parts are in binary format so they cannot be easily inspected by humans.

The datasets are **not** meant to be used/downloaded by end-users and are only relevant for internal testing purposes.
They are **not official** ASF release artifacts and they don't follow the ASF release life-cycle.

## Metastore database dumps

A collection of database dumps obtained from various versions of Hive.

#### Postgres

The database dumps were obtained from a Hive metastore running Cloudera's Hive 3.1.3000 using the following command:

    pg_dump -h localhost -p 5432 -U hive -d metastore -Fc > meta.dump.gz

Some manual post-processing has been performed to remove redundant information from the dump using the [cleanup.sql](metastore/cleanup.sql) file.
Nevertheless, the dumps may still contain additional databases and tables but these are not actively used.

##### TPC-DS

Metastore dumps from various [TPC-DS](http://www.tpc.org/tpcds/) scale factors.
* [metastore_tpcds10tb_3_1_3000](https://github.com/apache/hive-test-datasets/releases/download/1.0/metastore_tpcds10tb_3_1_3000.dump.gz)
* [metastore_tpcds30tb_3_1_3000](https://github.com/apache/hive-test-datasets/releases/download/1.0/metastore_tpcds30tb_3_1_3000.dump.gz)

## Iceberg 

### Metadata

A collection of Iceberg table directories holding only metadata and statistics (no data files) meant to be used mainly in planner tests.

#### TPC-DS

Metadata dumps from various [TPC-DS](http://www.tpc.org/tpcds/) scale factors.

##### 10TB metadata from S3

Metadata and stats obtained by downloading the content of the following S3 bucket (excluding data files and unrelated content): 
```
export AWS_ACCESS_KEY_ID=****************
export AWS_SECRET_ACCESS_KEY=************

aws s3 sync s3://dw-team-bucket/data/warehouse/tablespace/external/hive/tpcds_partitioned_iceberg_parquet_10000.db/ iceberg_s3_tpcds10tb --exclude "*" --include "*metadata*"
cd iceberg_s3_tpcds10tb
zip -r ../iceberg_s3_tpcds10tb.zip .
```

The DDL statements for querying the tables through Hive can be obtained by running the following script inside the downloaded directory.

```bash
for f in `ls`; do \
  metafile=`ls $f/metadata/*metadata.json | sort | tail -1 | sed 's|.*metadata/||'`; \
  echo "CREATE EXTERNAL TABLE $f STORED BY 'org.apache.iceberg.mr.hive.HiveIcebergStorageHandler' TBLPROPERTIES('metadata_location'='s3://dw-team-bucket/data/warehouse/tablespace/external/hive/tpcds_partitioned_iceberg_parquet_10000.db/$f/metadata/$metafile');"; \
done
```

WARNING: The script arbitrarily picks the *current* metadata file based on alphanumeric ordering. Ideally, this information should be obtained from the respective catalog.

Download: [iceberg_s3_tpcds10tb](https://github.com/apache/hive-test-datasets/releases/download/1.0/iceberg_s3_tpcds10tb.zip)


# Object store of the lakehouse. Iceberg data and metadata (Parquet, Avro,
# JSON) live in a bucket here; Lakekeeper vends per-request STS credentials
# for it to query engines.
#
# Pinned to the last MinIO community release with a maintained image.
FROM minio/minio:RELEASE.2025-09-07T16-13-09Z

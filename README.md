# test
from delta.tables import *
from pyspark.sql.functions import *
from pyspark.sql.types import *


def generate_table(bronze_table):
  table_path = f"{source_s3}/{bronze_table}/"
  dataframe = spark.read.format('parquet').load(table_path)
  target_partition=['cell_id'] if 'cell_id' in dataframe.columns else []
  except_column_lst = ['Op','transact_user'] if 'Op' in dataframe.columns else ['transact_user']
  temp_table_name = f"temp_{bronze_table}"

  def create_temp_table():
    return (
      spark.readStream.format("cloudFiles")
        .schema(dataframe.schema)
        .option("cloudFiles.format","parquet")
        .load(table_path)
    )
   
  dlt.create_streaming_live_table(
      name = bronze_table,
      table_properties = {
        "delta.autoOptimize.autoCompact": "true",
        "delta.autoOptimize.optimizeWrite": "true",
        "delta.enableChangeDataFeed": "true",
        "delta.isolationLevel": "Serializable"},
      partition_cols = target_partition
    )
  dlt.apply_changes(
        target = bronze_table,
        source = temp_table_name,
        keys = [col("id")],
        sequence_by = col("transact_seq"),
        ignore_null_updates = False,
        apply_as_deletes = "transact_operation = 'DELETE'",
        except_column_list = except_column_lst,
        stored_as_scd_type = 1
    )
    
    [generate_table(i) for i in list_tables ]

1. Enviar os dados para o hdfs
- descompactei e aloquei os arquivos em uma pasta chamada "projeto" dentro da pasta input
- docker exec -it namenode bash
- hdfs dfs -ls /user/marina/data
  - para verificar se a pasta ainda existe
- hdfs dfs -put /input/projeto /user/marina/data
2. Otimizar todos os dados do hdfs para uma tabela Hive particionada por 
município.
```python
spark.conf.set("hive.exec.dynamic.partition", "true")
spark.conf.set("hive.exec.dynamic.partition.mode", "nonstrict")
read_project = spark.read.csv("/user/marina/data/projeto/", header=True, sep=";",)
read_project.write.mode('overwrite').partitionBy('municipio').format('parquet').option('path',"/user/hive/warehouse/desafio_semantix").saveAsTable("p_municipio")
```
3. Criar as 3 vizualizações pelo Spark com os dados enviados para o HDFS
```python
from pyspark.sql.types import *
from pyspark.sql.functions import *
#view1:
view_t = df_view.withColumn("data", from_unixtime(unix_timestamp(col("data"), "yyyy-MM-dd"),"dd-MM-yyyy")).select("regiao", "estado", "municipio", "data", "semanaEpi", "populacaoTCU2019", "casosAcumulado", "casosNovos", "obitosAcumulado", "obitosNovos")
view_t2 = view_t.withColumn("populacaoTCU2019", col("populacaoTCU2019").cast(IntegerType())).withColumn("casosAcumulado", col("casosAcumulado").cast(IntegerType())).withColumn("semanaEpi", col("semanaEpi").cast(IntegerType())).withColumn("casosNovos", col("casosNovos").cast(IntegerType())).withColumn("obitosAcumulado", col("obitosAcumulado").cast(IntegerType())).withColumn("obitosNovos", col("obitosNovos").cast(IntegerType()))
view_1 = view_t2.sort(desc("data"))
#view2:
view_2 = df_view.groupBy("regiao").agg(format_number(avg(col("casosAcumulado").cast(IntegerType())),2).cast(IntegerType()).alias("mediaCasosAcumulado"),format_number(stddev(col("casosAcumulado").cast(IntegerType())),2).cast(IntegerType()).alias("desvioPadraoCasosAcumulado"), format_number(avg(col("obitosAcumulado").cast(IntegerType())),2).cast(IntegerType()).alias("mediaObitosAcumulado"),format_number(stddev(col("obitosAcumulado").cast(IntegerType())),2).cast(IntegerType()).alias("desvioPadraoObitosAcumulado"))
#view3:
view_3 = df_view.groupBy("regiao").agg(format_number(avg(col("casosAcumulado").cast(IntegerType())),2).alias("mediaCasosAcumulado"))
kafka_view_3 = view_3.withColumnRenamed("regiao", "key").withColumnRenamed("mediaCasosAcumulado", "value")
kafka_view_3.write.format("kafka").option("kafka.bootstrap.servers", "kafka:9092").option("topic", "topic_view3").save()
```
4. Salvar a primeira visualização como tabela Hive
```python
view_1.write.saveAsTable("table_view_1")
```
5. Salvar a segunda visualização com formato parquet e compressão snappy
```python
view_2.write.parquet("/user/marina/data/view_2")
```
6. Salvar a terceira visualização em um tópico no Kafka
```python
view_1.write.saveAsTable("table_view_1")
```
7. Criar a visualização pelo Spark com os dados enviados para o HDFS
8. Salvar a visualização do exercício 6 em um tópico no Elastic
9. Criar um dashboard no Elastic para visualização dos novos dados enviados

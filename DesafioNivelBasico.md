1. Enviar os dados para o hdfs
- descompactei e aloquei os arquivos em uma pasta chamada "projeto" dentro da pasta input
```
docker exec -it namenode bash
hdfs dfs -ls /user/marina/data
##### para verificar se a pasta ainda existe
hdfs dfs -put /input/projeto /user/marina/data
```
2. Otimizar todos os dados do hdfs para uma tabela Hive particionada por 
município.
```python
spark.conf.set("hive.exec.dynamic.partition", "true")
spark.conf.set("hive.exec.dynamic.partition.mode", "nonstrict")
read_project = spark.read.csv("/user/marina/data/projeto/", header=True, sep=";",)
read_project.write.mode('overwrite').partitionBy('municipio').format('parquet').option('path',"/user/hive/warehouse/desafio_semantix").saveAsTable("p_municipio")
```
- ![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/3e4f2cf3-743c-4fd9-9d7e-3a28b579a53e)
- ![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/b793df60-6009-4aa5-bd49-ebad381da7e2)
- ![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/843f661f-cea5-4e2b-bc5c-6a962295965b)


3. Criar as 3 vizualizações pelo Spark com os dados enviados para o HDFS
```python
from pyspark.sql.types import *
from pyspark.sql.functions import *
df_view = spark.read.table("p_municipio")
#view1:
view_t = df_view.withColumn("data", from_unixtime(unix_timestamp(col("data"), "yyyy-MM-dd"),"dd-MM-yyyy")).select("regiao", "estado", "data", "semanaEpi", "casosAcumulado", "casosNovos", "obitosAcumulado", "obitosNovos")
view_t2 = view_t.withColumn("casosAcumulado", col("casosAcumulado").cast(IntegerType())).withColumn("semanaEpi", col("semanaEpi").cast(IntegerType())).withColumn("casosNovos", col("casosNovos").cast(IntegerType())).withColumn("obitosAcumulado", col("obitosAcumulado").cast(IntegerType())).withColumn("obitosNovos", col("obitosNovos").cast(IntegerType()))
view_1 = view_t2.sort(desc("data"))
#view2:
view_2 = df_view.groupBy("regiao").agg(format_number(avg(col("casosAcumulado").cast(IntegerType())),2).cast(IntegerType()).alias("mediaCasosAcumulado"),format_number(stddev(col("casosAcumulado").cast(IntegerType())),2).cast(IntegerType()).alias("desvioPadraoCasosAcumulado"), format_number(avg(col("obitosAcumulado").cast(IntegerType())),2).cast(IntegerType()).alias("mediaObitosAcumulado"),format_number(stddev(col("obitosAcumulado").cast(IntegerType())),2).cast(IntegerType()).alias("desvioPadraoObitosAcumulado"))
#view3:
view_3 = df_view.groupBy("regiao").agg(format_number(avg(col("casosAcumulado").cast(IntegerType())),2).alias("mediaCasosAcumulado"))
```
4. Salvar a primeira visualização como tabela Hive
```python
view_1.write.saveAsTable("table_view1")
```
- **View 1**
![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/9956dc00-d006-4199-939f-1da58e98b673)
![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/e7a27113-1173-4a92-8f33-d2b02ad8d66c)

5. Salvar a segunda visualização com formato parquet e compressão snappy
```python
view_2.write.parquet("/user/marina/data/view_2")
```
- **View 2**
  ![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/e1281fff-1834-4db3-b9d5-ea4685a2086f)
  ![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/939af2f8-a8bf-4221-bd69-4ccbc92a3f61)
  
6. Salvar a terceira visualização em um tópico no Kafka
```python
kafka_view_3 = view_3.withColumnRenamed("regiao", "key").withColumnRenamed("mediaCasosAcumulado", "value")
kafka_view_3.write.format("kafka").option("kafka.bootstrap.servers", "kafka:9092").option("topic", "topic_view3").save()
```
- **View 3**
  ![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/8e131b74-a9a5-4e4e-8585-bd89cb40d8e3)

7. Criar a visualização pelo Spark com os dados enviados para o HDFS
- **View 1**
![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/acf35726-d291-4f85-a6d8-1f68b017799b)
- **View 2**
![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/0e545166-6e49-43cf-a612-3d825439a565)
- **View 3**
  ![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/d65b47db-eeae-48ab-a339-5de314f0df8e)


8. Salvar a visualização do exercício 6 em um tópico no Elastic
9. Criar um dashboard no Elastic para visualização dos novos dados enviados

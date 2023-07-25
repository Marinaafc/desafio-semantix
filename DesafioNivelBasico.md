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
4. Salvar a primeira visualização como tabela Hive
5. Salvar a segunda visualização com formato parquet e compressão snappy
6. Salvar a terceira visualização em um tópico no Kafka
7. Criar a visualização pelo Spark com os dados enviados para o HDFS
8. Salvar a visualização do exercício 6 em um tópico no Elastic
9. Criar um dashboard no Elastic para visualização dos novos dados enviados

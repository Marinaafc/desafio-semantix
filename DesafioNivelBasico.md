1. Enviar os dados para o hdfs
- Ex_Aula-2a
- cd input
- https://livreeaberto.com/baixar-arquivos-do-terminal-linux
- docker exec -it namenode bash
- hdfs dfs -ls /user/marina/data
- Se não existir, criar
- hdfs dfs -mkdir -p /user/marina/data
- hdfs dfs -put /input/* /user/marina/data
- Visualizar aula 3 de Big Data Foundations qualquer coisa
2. Otimizar todos os dados do hdfs para uma tabela Hive particionada por 
município.
- Exercício Aula 4 e 5 - Hive Básico
3. Criar as 3 vizualizações pelo Spark com os dados enviados para o HDFS
4. Salvar a primeira visualização como tabela Hive
5. Salvar a segunda visualização com formato parquet e compressão snappy
6. Salvar a terceira visualização em um tópico no Kafka
7. Criar a visualização pelo Spark com os dados enviados para o HDFS
8. Salvar a visualização do exercício 6 em um tópico no Elastic
9. Criar um dashboard no Elastic para visualização dos novos dados enviados

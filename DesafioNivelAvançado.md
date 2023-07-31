# Parte 1 - Consumir os dados da API de Elasticsearch
## Tentativa 1 - Consumo através do Elasticsearch-Hadoop
- Tendo como base os links disponibilizados no Desafio, minha primeira tentativa foi pelo Elasticsearch-Hadoop.
1. Primeiramente acessei o container do Jupyter-Spark para acessar a dependência do Elasticsearch-Spark:
```
docker exec -it jupyter-spark bash
spark-shell --packages org.elasticsearch:elasticsearch-spark-20_2.11:8.9.0
```
2. Depois fiz importação e criei uma sessão no Spark, definindo em seguida as configurações necessárias para carregar os dados em um DataFrame:
```scala
import org.apache.spark.sql.SparkSession
val spark = SparkSession.builder.appName("ElasticSearch").getOrCreate()
val esConfig = Map("es.nodes" -> "https://imunizacao-es.saude.gov.br", "es.net.http.auth.user" -> "imunizacao_public", "es.net.http.auth.pass" -> "qlto5t&7r_@+#Tlstigi", "es.resource.read" -> "desc-imunizacao-v5", "es.nodes.wan.only" -> "true", "es.port" -> "443")
val df = spark.read.format("org.elasticsearch.spark.sql").options(esConfig).load()
```
3. Porém, me deparei com um problema, pois o usuário disponibilizado não tinha autorização para acessar os índices da API e eu precisava obrigatoriamente definir um índice para utilizar o "load()" e ler os dados no DataFrame
![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/31284206-ba8f-4cc8-83ce-339dba0925e0)
![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/5606c120-0dcf-49bf-9836-442265fd33ff)


## Tentativa 2 - Consumo pelo Logstash
- Após passar bastante tempo procurando outra forma de consumir os dados pelo Elasticsearch-Hadoop sem sucesso, tive a ideia de seguir o que estava posto no manual e realizar esse consumo pelo Postman.
- No entanto, depois de refletir um pouco, percebi que não seria algo coerente com o propósito do desafio, pois os dados não seriam atualizados em tempo real como é nas visualizações do site que devem ser replicadas.
- Por conta disso, após pesquisar mais um pouco, encontrei a possibilidade de consumo em tempo real da API pelo Logstash.
1. Para que fosse possível o consumo pelo Logstash, precisei alterar o arquivo logstash.conf
```
input {
  http_poller {
    urls => {
      test1 => {
      method => get
      user => "imunizacao_public"
      password => "qlto5t&7r_@+#Tlstigi"
      url => "https://imunizacao-es.saude.gov.br/_search"
      headers => {
        Accept => "application/json"
      }
    }
  }
  request_timeout => 60
  schedule => { cron => "* * * * * UTC"}
  codec => "json"
  metadata_target => "http_poller_metadata"
  }
}
output {
  stdout {
    codec => "json"
  }
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "desafio_api"
  }
}
```
2. E deu certo! Os dados começaram a chegar em um índice no Elasticsearch.
![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/72f3910e-29ec-4b44-8679-546027a70fbd)
![image](https://github.com/Marinaafc/desafio-semantix/assets/107056644/977dfcc0-b87d-4f9f-9a23-f3476c17457b)



# Parte 2 - Replicar as visualizações do site “https://covid.saude.gov.br/”
- Em desenvolvimento

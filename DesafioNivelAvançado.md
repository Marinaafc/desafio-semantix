# Parte 1 - Consumir os dados da API de Elasticsearch
## Tentativa 1 - Consumo através do Elasticsearch-Hadoop
- Tendo como base os links disponibilizados no Desafio, minha primeira tentativa foi pelo Elasticsearch-Hadoop.
1. Primeiramente acessei o container do Jupyter-Spark para acessar a dependência do Elasticsearch-Spark:
```

```
2. Depois fiz importação e criei uma sessão no Spark, definindo em seguida as configurações necessárias para carregar os dados em um DataFrame:
```

```
3. Porém, me deparei com um problema, pois o usuário disponibilizado não tinha autorização para acessar os índices da API e eu precisava obrigatoriamente definir um índice para utilizar o "load()" e ler os dados no DataFrame
-Imagens-
## Tentativa 2 - Consumo pelo Logstash
- Após passar bastante tempo procurando outra forma de consumir os dados pelo Elasticsearch-Hadoop sem sucesso, tive a ideia de seguir o que estava posto no manual e realizar esse consumo pelo Postman. No entanto, depois de refletir um pouco, percebi que não seria algo coerente com o propósito do desafio, pois os dados não seriam atualizados em tempo real como é nas visualizações do site que devem ser replicadas.
- Por conta disso, após pesquisar mais um pouco, encontrei a possibilidade de consumo em tempo real da API pelo Logstash.
1. Para que fosse possível o consumo pelo Logstash, precisei alterar o arquivo logstash.conf
Imagem
2. E deu certo! Os dados começaram a chegar em um índice no Elasticsearch.
Imagem

# Parte 2 - Replicar as visualizações do site “https://covid.saude.gov.br/”
- A parte que eu achei que seria mais tranquila de fazer, acabou sendo mais difícil. Depois que consegui acessar os dados da API, comecei a estranhar o fato de que não tinha nenhum campo que me oferecesse a possibilidade de calcular o nº de óbitos.
- Devido a essa dificuldade, ainda estou buscando uma forma de replicar as visualizações do site.

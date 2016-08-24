# ELK Stack

Este repositório utiliza Docker e Docker-compose para criar uma stack de ELK (Utilizando as imagens oficiais do Elasticsearch, Logstash e Kibana).  
Com ele você será capaz de analisar qualquer conjunto de dados usando os recursos de pesquisa e agregação do Elasticseach eo poder de visualização de Kibana.

### Imagens oficiais:
- [ElasticSearch](https://hub.docker.com/_/elasticsearch/)
- [Logstash](https://hub.docker.com/_/logstash/)
- [Kibana](https://hub.docker.com/_/kibana/)

### Requisitos
- Docker
- Docker Compose

### Utilizando este repositorio
1. **Clone o repositorio**
```bash
$ git clone <reponame>
```
2. **Iniciando o ELK utilizando o Docker compose**
```bash
$ docker-compose up
# Você também pode executa-lo em background
# docker-compose up -d
```
Agora que a pilha está em execução, você pode injetar registros na mesma. A configuração logstash permite enviar conteúdo via TCP.
```bash
# Exemplo de envio de conteudo para o Logstash
$ nc localhost 5000 < /path/to/logfile.log
```
3. **Acessando o Kibana**  
Você pode acessar o Kibana UI diretamente do seu navegador atraves do endereço http://localhost:5601.  
***NOTA:*** Você precisa injetar conteudo no logstash antes de poder criar um índice. Então tudo que você deve ter que fazer é apertar o botão de criar.

4. **Acessando o Sense**
Você também pode acessar o Sense atraves do endereço http://localhost:5601/app/sense.  
***NOTA:*** Para utilizar Sense, você precisará consultar o endereço IP associado ao dispositivo de rede em vez de localhost.

### Um pouco mais sobre esta pilha
#### Portas:
Por padrão, a pilha expõe os seguintes portas:  

| Porta |          Serviço        |
|-------|:-----------------------:|
| 5000  | Entrada TCP do logstash |
| 9200  | HTTP do elasticsearch   |
| 9300  | TCP do elasticsearch    |
| 5601  | Kibana                  |

***NOTAS:***
- Se você estiver usando boot2docker, você deve acessá-lo através do endereço de IP boot2docker no lugar do localhost.
- A configuração não é recarregada dinamicamente, você precisará reiniciar a pilha após qualquer mudança na configuração de um componente.

#### Configurações:
Você pode ajustar o do Kibana e logstash editando seus arquivos de de configurações.  

Kibana: kibana/config/kibana.yml  
Logstash: logstash/config/logstash.conf

O diretório logstash/config é mapeado sobre o /etc/logstash/conf.d do container permitindo a criação de multiplos arquivos de configuração, estes arquivos serão lidos em ordem alfabetica.
Você pode obter mais informações sobre como configurar o logstash lendo sua [documentação oficial](https://www.elastic.co/guide/en/logstash/2.3/index.html).

O container Logstash utiliza a variável de ambiente LS_HEAP_SIZE para determinar a quantidade de memória que será associada à memória heap JVM (o padrão é 500). Você pode ajusta-la no docker-compose.yml para que atenda melhor as suas necessidades.
```yml
# Exemplo de ajuste da memória do logstash
version: '2'

services:
  ...
  logstash:
    build: logstash/
    command: logstash -f /etc/logstash/conf.d/logstash.conf
    volumes:
      - ./logstash/config:/etc/logstash/conf.d
    ports:
      - "5000:5000"
    links:
      - elasticsearch
    environment:
    - LS_HEAP_SIZE=2048m
  ...
```

Você também pode istalar plugins ao logstash, basta adicinar as instruções necessárias ao arquivo logstash/Dockerfile.

```plantext
# Exemplo de instalação de plugin ao logstash
FROM logstash:latest

RUN logstash-plugin install logstash-input-s3
```
Em seguida basta adicionar as configurações do plugin ao arquivo config/logstash.conf

```plantext

s3{
     access_key_id => "aws_key"
     secret_access_key => "aws_access_key"
     region => "eu-west-1"
     bucket => "my_bucket"
     size_file => 2048
     time_file => 5
   }
```

O container ElasticSearch está usando as configurações padrões, mas isto não impede que você as ajustes para atender suas necessidades. Para isto basta criar o arquivo  elasticsearch/config/elasticsearch.yml e adicionar as configurações a ele e em seguida ajustar seu docker-compose.yml.

```yml
# Exemplo de ajuste do elasticsearch no docker-compose.yml
version: '2'

elasticsearch:
  build: elasticsearch/
  command: elasticsearch -Des.network.host=_non_loopback_
  ports:
    - "9200:9200"
  volumes:
    - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml

...
```

Os dados armazenados pelo elasticsearch sã mantidos após a reinicialização do container, porém são removidos após a remoção do container.  
Caso haja a necessidade de manter estes dados você pode montar um volume para o armazemanto destas informações.

```yml
# Exemplo de armazenamento
version: '2'
elasticsearch:
  build: elasticsearch/
  command: elasticsearch -Des.network.host=_non_loopback_
  ports:
    - "9200:9200"
  volumes:
    - /path/to/storage:/usr/share/elasticsearch/data
    ...
```

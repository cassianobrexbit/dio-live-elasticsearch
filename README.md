# dio-live-elasticsearch
Repositório de código para a live sobre o Amazon Opensearch (sucessor do Elasticsearch)

## Serviços utilizados
- EC2 (servidor de logs)
- Kinesis Data Firehose (stream de dados)
- Elasticsearch (pesquisa e visualização dos dados)

## Configurações

### Iniciar instância no EC2
- EC2 Console -> Launch instances -> Amazon Linux 2 AMI (HVM), SSD Volume Type -> t2.micro -> Review and launch -> Create a new key pair -> download da chave ssh
- Acessar com aplicação SSH
- Utilizar o comando ```wget``` para obter os arquivos de logs que estão no seguinte link: https://dio-live-elasticsearch-data.s3.amazonaws.com/httpd.zip
- Descompactar o arquivo .zip com o comando ```unzip httpd.zip```
- Mover a pasta descompactada com o comando ```sudo mv htppd /var/log/httpd```

#### Configurar o Kinesis Agent
- ```yum install aws-kinesis-agent```

### Configurar cluster do Elasticsearch

- Open Search (sucessor do Elasticsearch) -> Create new domain  -> Type: development and testing -> version 7.10 -> Next -> Domain name [seu_nome_de_dominio] -> manter o restante das configurações como padrão -> Next
- Selecionar Public Access -> Desabilitar Fine grained access control -> Access Policy -> Custom Access Policy -> IPV4 address [sei IP] -> Allow
- Manter o restante das configurações padrão -> Next -> Create Domain

### Configurar o Kinesis Data Stream

- Kinesis Dashboard -> Create new delivery stream
  - Source -> Direct PUT
  - Destination -> Amazon Elasticsearch Service
  - Delivery stream name -> [DIOLiveStream]
  - Transform records -> Enabled -> Create function -> Blueprint Apache Log to JSON -> Use blueprint
  - Table name parameter -> weblogs
  - Deploy
  - OBS: utiliza o Cloudformation
  - Atualizar o timeout da função lambda para 1 minuto
- Destination
  - Elasticsearch domain (aguardar o processamento do domínio)
  - Index -> weblogs
  - Rotation -> Every day
- S3 Backup
  - Create new bucket
- S3 bucket prefix ```es/```

### Configurar delivery stream no EC2

- ```sudo nano /etc/aws-kinesis/agent.json```
```
{
  "cloudwatch.emitMetrics": true,
  "kinesis.endpoint": "",
  "firehose.endpoint": "firehose.us-east-1.amazonaws.com",

  "awsAccessKeyId":"",
  "awsSecretAccessKey":"",

  "flows": [
    {
      "filePattern": "/var/log/httpd/ssl_access*",
      "deliveryStream": "",
      "initialPosition": "START_OF_FILE"
    }
  ]
}
```
- ```sudo service aws-kinesis-agent restart```
- ```tail -f /var/log/aws-kinesis-agent/aws-kinesis-agent.log```

### Elasticsearch

- Indices
- Overview -> Link to Kibana
- Acessar o Kibana
  - Manage -> Index patterns -> Create index pattern [weblogs*] -> Time field [@timestamp]
  - Pesquisar dados
    - Discover -> configurar datas para 2019 - date absolute 27/01/2019 02/02/2019
    - Visualization -> New -> Vertical bar -> Filter [Response code 500 is] 
    - Bucket -> Aggregation Date Histogram -> Field @timestamp -> Minimum interval Hour -> Update

### Limpar recursos

- Deletar Delivery Stream
- Deletar domain do Elasticsearch
- Deletar função lambda
- Deletar bucket do S3
- Encerrar instância no EC2

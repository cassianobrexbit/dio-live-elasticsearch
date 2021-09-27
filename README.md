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
- Utilizar o comando ```wget``` para obter os arquivos de logs que estão [neste repositório](https://dio-live-elasticsearch-data.s3.us-east-1.amazonaws.com/httpd.zip?response-content-disposition=attachment&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEBgaCXVzLWVhc3QtMSJHMEUCIE%2FOMbPUsZ9YmDoRhrbwvJ1OKuLTyuj4jeUsvIku%2F4zWAiEA2aSCXxsjlH6GKxqmQf2wdjR76DvjGDzrfh644IZZw%2FEqlgMIgf%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARAAGgwwMDcxMzM2NTQ0MTYiDIU%2FA7M%2F8WgcOBYqlCrqAoRnA%2B67CFIL%2FpKbwa%2Fcqrstuqh49XZAxdw8XTufBU0vYONQpLQ%2FXGID6TEG42%2FaW9tFKGG%2BmSdASa7qsPsZHZ1H7kNPlYMvreOe0lSMaNVqxpfvyT7NuaoPZ6SteCIblByYdMWxKlQIoXDaXqgSKf8t9J0DRk7XMStXSQ4C14lbToVNHtGOL8chbph9ZnpPiGgAcnrcbt35i%2B9erO8pjthbI%2FDlZI7pv1y9S1AUGOiMmuDHgFp0RGZcpM23eMx7Ze%2FPgRtCLU30I18yPFirjVl621UqrE9xzk4gHJcwHpHzoHU463QVKr4tajct4EW8YfRrKykutEn4DajpcyTukmYFo2ZLB5tZAriTtknhBny2lVXf4KuUhpgDI3Wxy185Nlz%2Bmx7xjl06%2BuT9khjTo%2BJNW4blFu1wR1o5u2XPYBVe0K2d71H2oZZ7d6ylbEtmL7QS79W5HNC9pX2HitcNvOqWA%2FcHUJnIs4XTMJ2ewooGOrMCxroBSTSmz2oTlUgFPh4Osw8z1L%2Fz9z39cSIuQA5GwqLMOHgTlOzE4YJJZ043VI3TllmXZHrICdouubEm1lPywBawlUcPQ6e1bWtWU%2BV51XR%2F775jyd2PcZBeeOQkYet8V04bbH1nPW1AbhGW8yndMZnkPmXums7t4p3rU6kxb54LPgVSEn1Zz5fo%2FInsk8LqaaK%2F71EwBMG%2FVCX%2FcVvygyvCXoPKssH1sAXf508LTMcCHMJsNeY9CfyvdGoUowmaa2c6cwyWOCLOF2oYrszulohjWKPHGH7HnfJBcxosydb4mvpr%2F1TbI40yf%2FFxsVBpmGHunUdSCb5%2FU8r%2BxOE8L9Ur7sBUculdfrDSE%2B4%2Fy1%2FO281kDyHlrkReF2BgdLY23wL9IVNtXkupGhJfpTbECvYPPQ%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20210927T002750Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAQDKJS5WIOEGCZVIZ%2F20210927%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=dafe217355dacd45db76598b0c94559412ce999455909b6e31283d4fba00002d)
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

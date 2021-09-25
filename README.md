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
- Utilizar o comando ```wget``` para obter os arquivos de logs que estão neste repositório ```httpd.zip```
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

# Introdução ao Terraform

Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions.  
Configuration files describe to Terraform the components needed to run a single application or your entire datacenter. Terraform generates an execution plan describing what it will do to reach the desired state, and then executes it to build the described infrastructure. As the configuration changes, Terraform is able to determine what changed and create incremental execution plans which can be applied.
* A maioria dos IaC são descritivos, e.g. K8s e Terraform. O modo imperativo é comando por comando, com todos os detalhes, ifs e elses.
* O Terraform é usado para SDN e outros exemplos aqui: https://www.terraform.io/intro/use-cases.html.
* Packing: Construção de imagens.
* Infra imutável: Utiliza descrição declarativa.

# Arquivo HCL
https://www.terraform.io/docs/configuration/syntax.html

## Blocos
É como um bloco de código, mas com tipos específicos de bloco (resource) e tipos específicos de tipos. Abaixo tem o exemplo de um bloco.
```
resource "aws_instance" "example" {
  ami = "abc123"

  network_interface {
    # ...
  }
}
```
* Resourse: "aws_instance".
* Name: "example".
* Argumento chave-valor: ami = "abc123".
* Qualquer arquivo do terraform precisa trabalhar com dois tipos de blocos: provider e terraform.

 
## Identificadores
Qualquer identificador dentro do bloco, pode ser: nome de argumento; nome de bloco.  
Eles podem conter underscore (_) e hyphens (-). O primeiro caractere de um identificador não pode ser um número, para evitar ambiguidade em achar que o número literal é um inteiro.

## Argumentos
Argumentos que ficam dentro do bloco. 

### Provider 
Manipula o provedor do terraform e.g. AWS e GCP.  
Cada provider tem as suas configurações específicas que podem ser vistas aqui: https://www.terraform.io/docs/providers/index.html. 

### Inicialization
Após configurar o provider é hora de inicializa-lo. Aqui é passado as credenciais, que vão possibilitar que o terraform interaja com o provider específico.  
No AWS o gerenciamento de credenciais é feito via Identity and Access Management (IAM).  
Dentro do IAM, deve ser criado um usuário e dentro dele definir as permissões. Após a criação do usuário, será visto o ID da chave de acesso e a chave de acesso secreta. As credenciais podem ser usadas de diversas formas e uma delas é via variável de ambiente, que é o que sera usado aqui.  
O terraform não será instalado na máquina, será usado o docker, onde o diretório atual deve conter os arquivos configurados por você, são eles: main.tf, ec2.tf, variable.tf e output.tf.  
* No main.tf é necessário alterar o nome do bucket, mais informações em: https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html.
O comando para executar o container:
```
docker run -it -v $PWD:/app -w /app --entrypoint "" --name terraform hashicorp/terraform:light sh
```
O terminal aberto já é o do container e pode-se verificar a versão do terraform, para confirmar que tudo está dentro do esperado: 
```
terraform version
```
As credenciais precisam ser inseridas, elas já foram criadas no IAM. A inserção no container é feito por meio de variáveis de ambiente.  
```
export AWS_ACCESS_KEY_ID="tururu" #ID da chave de acesso
export AWS_SECRET_ACCESS_KEY="tururu" #Chave de acesso secreta
```
Para começar a trabalhar com o terraform:
```
terraform init
```
Este comando é responsável por configurar a pasta para habilitar o uso do terraform.






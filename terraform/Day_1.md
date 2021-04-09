___
# Introdução ao Terraform - Aula 1

Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions.  
Configuration files describe to Terraform the components needed to run a single application or your entire datacenter. Terraform generates an execution plan describing what it will do to reach the desired state, and then executes it to build the described infrastructure. As the configuration changes, Terraform is able to determine what changed and create incremental execution plans which can be applied.
* A maioria dos IaC são descritivos, e.g. K8s e Terraform. O modo imperativo é comando por comando, com todos os detalhes, ifs e elses.
* O Terraform é usado para SDN e outros exemplos aqui: https://www.terraform.io/intro/use-cases.html.
* Packing: Construção de imagens.
* Infra imutável: Utiliza descrição declarativa.

___
# Arquivo HCL - Aula 2
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
As credenciais precisam ser inseridas tanto diretamente no container quanto no main.tf, elas já foram criadas no IAM. No caso do container, as credenciais são inseridas em variáveis de ambiente.  
```
export AWS_ACCESS_KEY_ID="tururu" #ID da chave de acesso
export AWS_SECRET_ACCESS_KEY="tururu" #Chave de acesso secreta
```
Para começar a trabalhar com o terraform:
```
terraform init
```
Este comando é responsável por configurar a pasta para habilitar o uso do terraform.  
Em caso de erro, tente:
* Se o bucket foi criado e o nome está correto.
* Apagar o .terraform gerado e repetir o processo.
* Refazer a autenticação IAM e repetir o processo.

___
# Comandos básicos - Aula 3
Iniciando pela definição de backend, temos:
A "backend" in Terraform determines how state is loaded and how an operation such as apply is executed. This abstraction enables non-local file state storage, remote execution, etc.  
O backend é definido no main.tf, caso não seja definido nenhum backend, será usado o local backend. Os arquivos backend serão guardados na própria máquina que está executando o tarraform. Um dos problemas de ter o local backend é que todo mundo que vai usar terraform no time, precisa ter acesso a este arquivo.  
Uma alternativa a esse problema é o backend remoto, um exemplo está mostrado abaixo.
```
terraform { # Não precisa colocar tipo do bloco nem nome
  backend "s3" {
    bucket = "name_bucket" # Bucket criado no s3, ele não deve ser público
    key    = "name_key"
    region = "name_region"
  }
}
```

1. terraform init  
Já foi falado sobre ele, é o responsável por baixar os plugins dos providers, inicia tudo.  
2. terraform plan -out plano  
O terraform gera um plano baseado no que esta descrito no HCL, este plano é enviado para o arquivo "plano" que é um arquivo binário.  
A saída deste comando lista tudo de novo que é criado, no início de cada linha do que foi criado tem um + na cor verde. 
Caso a mudança acarrete em algo deletado será mostrado um - na cor vermelha, caso seja uma alteração será mostrado um ~ amarelo.
3. terraform apply "plano" 
Este comando vai lê o arquivo "plano" criado anteriormente e compara com o arquivo de estado, o "terraform.tfstate", a diferença entre eles será aplicado como recurso, na AWS. Inicialmente o arquivo de estado estava em branco, logo, tudo que estava no arquivo do plano foi gerado como recurso e o arquivo "terraform.tfstate" foi atualizado.  
Depois que esse comando finaliza, o arquivo de estado é alterado. Este arquivo é sensível e não deve ser comitado no github.
4. terraform destroy  
Verifica tudo que está no estado e no provider, em seguida destrói tudo.

___
# Expressions and Console - Aula 4
## Expressions
https://www.terraform.io/docs/configuration/expressions.html  
Todos os valores ou referências que serão usadas para se referir ou calcular valores dentro de uma configuração. Podem ser classificados em:
* string: "hello".
* number: 15 or 15.266565.
* bool: true or false.
* list (or tuple): ["us-west-1a", "us-west-1c"]. 
* map (or object): {name = "Mabel", age = 52}.
* null.

### Indices 
* local.list[3] 
* Para todos os índices: local.list[*].

### References
São vários tipos de valores nomeados, uma expressão que faz referência ao valor associado; você pode usá-los como expressões independentes ou combiná-los com outras expressões para calcular novos valores.  
* Recurso: ```<RESOURCE TYPE>.<NAME>``` Qualquer referência, que não se encaixe em algum tipo abaixo, será entendido como um recurso.
* Variáveis de ambiente: ```var.<NAME>```
* Informações locais: ```local.<NAME>```
* Módulos: ```module.<MODULE NAME>.<OUTPUT NAME>```
* Data, pega o que está na cloud e guarda no estado: ```data.<DATA TYPE>.<NAME> ```  
* ```path.module ```
* ```path.root ```
* ```path.cwd```
* ```terraform.workspace```

## Console
This command provides an interactive command-line console for evaluating and experimenting with expressions. This is useful for testing interpolations before using them in configurations, and for interacting with any values currently saved in state.  
Ou seja, o console possibilita a iteração de testes com os valores salvos no estado, seja local ou remoto.  
Para executar o console: ```terraform console [options] [dir]```.
* Vamos praticar:   
Iniciando o console: ```terraform console```.  
```aws_instance.web.attribute``` Isso vai acessar o recurso e mostrar todas as informações sobre ele.  
Para sair do console: Ctrl + C ou Ctrl + D.

### Atributos
Todo recurso cria valores de dois tipos.  
O primeiro são os argumentos, onde a combinação de chave-valor é inserida na criação do recurso e foram setadas por você.   
O segunto tipo, os atributos, estão ligados aos valores de retorno, que são valores inseridos após a aplicação do recurso no provider. São conhecidos como atributos.  

### Built-in functions
The Terraform language includes a number of built-in functions that you can call from within expressions to transform and combine values. The general syntax for function calls is a function name followed by comma-separated arguments in parentheses: ```max(5, 12, 9)```.

#### cidrsubnet
Calculates a subnet address within given IP network address prefix.
```cidrsubnet(prefix, newbits, netnum)```
```newbits``` is the number of additional bits with which to extend the prefix. For example, if given a prefix ending in /16 and a newbits value of 4, the resulting subnet address will have length /20.
```netnum``` is a whole number that can be represented as a binary integer with no more than newbits binary digits, which will be used to populate the additional bits added to the prefix.

___
# Providers - Aula 5
https://www.terraform.io/docs/configuration/providers.html  
While resources are the primary construct in the Terraform language, the behaviors of resources rely on their associated resource types, and these types are defined by providers.  
Link com os recursos a serem criados, dentro do destino.  
É um bloco do tipo provider
```
provider "google" {
  project = "acme-app"
  region  = "us-central1"
} 
```
Os argumentos são específicos do tipo do provider escolhido. Mas existem dois argumentos que são próprios do terraform e são compatíveis com todos os providers, são os meta-argumentos.
Especifica a versão a ser trabalhada: ```version```.  
Assegura que o comportamento dos plugins com o provider estão dentro do que você espera em determinada versão.  
Possibilidade de criar instâncis em múltiplos providers, ou ainda o mesmo provider com configurações diferentes: ```alias```. Vide exemplo abaixo:
```
# The default provider configuration
provider "aws" {
  region = "us-east-1"
}

# Additional provider configuration for west coast region
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}
```
## Plugins de terceiros
https://www.terraform.io/docs/configuration/providers.html#third-party-plugins  
Devem ser inseridos em ```~/.terraform.d/plugins```, quando for dado o ```terraform init```, o terraform será inicializado, mas não vai baixar anda porque o plugin já está na pasta.

___
# Variáveis - Aula 6
https://www.terraform.io/docs/configuration/variables.html  
Possibilita variar os valores dentro do HCL. Tem o objetivo de tornar o codigo modular, dado a mudança de uma variável.  
```
variable "image_id" {
  type = string
}
```
O nome da variável pode ser qualquer um, exceto: source, version, providers, count, for_each, lifecycle, depends_on e locals.  
Deve ser definido o ```type```, caso não coloque será gerado um erro.  
Exemplo de uso da variável ```image_id``` criada anteriormente:
```
resource "aws_instance" "example" {
  instance_type = "t2.micro"
  ami           = var.image_id
}
```
* default: Dentro de cada variável pode-se definir o valor default, que é o valor padrão atribuído a variável em questão.  
* description: Descrição da variável.

## Formas de setar variáveis
1. Variables on the Command Line  
To specify individual variables on the command line, use the ```-var``` option when running the ```terraform plan``` and ```terraform apply``` commands:
``` terraform plan -var image_id="ami-abc123" -out plan ```  
Nesse caso é alterado o iam id, a máquina será destruída e recriada, caso queira desfazer essa alteração, basta rodar o comando ``` terraform plan -out plan ``` que a variável image_id irá pegar o valor default.  
Ainda é possível usar o: ```terraform apply -var="image_id=ami-abc123"```.  
2. Variable Definitions (.tfvars) Files
To set lots of variables, it is more convenient to specify their values in a variable definitions file (with a filename ending in either .tfvars or .tfvars.json) and then specify that file on the command line with -var-file: ```terraform apply -var-file="testing.tfvars"```.
A variável deve ser da seguinte forma: ```image_id = "ami-abc123"```.  
Terraform also automatically loads a number of variable definitions files if they are present: 
* Files named exactly terraform.tfvars or terraform.tfvars.json.
* Any files with names ending in .auto.tfvars or .auto.tfvars.json.
3. Environment Variables
Terraform searches the environment of its own process for environment variables named TF_VAR_ followed by the name of a declared variable.
```export TF_VAR_image_id=ami-abc123```

### Variable Definition Precedence
Ordem crescente de precedência
* default
* ```terraform.tfvars```
* ```terraform.tfvars.json```
* ```*.auto.tfvars``` or ```*.auto.tfvars.json```
* ```var``` and ```-var-file ```

____
# Execução remota ou local - Aula 7

## Provisoners (Gerência de configuração, provisionamento) != Providers (Integração com cloud usado pelo terraform)
https://www.terraform.io/docs/provisioners/index.html  
Não é recomendado usar o provisoners do terraform sempre, apenas em último caso.  
* Para passar algum dado assim que a máquina é iniciada, pode-se usar um provisoner, mas já existem algumas ferramentas das próprias empresas que hospedam as nuvens com esse fim: https://www.terraform.io/docs/provisioners/index.html#passing-data-into-virtual-machines-and-other-compute-resources.  
* No caso de máquinas com configuraçõs genéricas, o Terraform aconselha o uso de imagens personalizada, com o Packer, por exemplo.

## How to use Provisioners
Os Provisioners estão ligados a criação de um recurso, normalmente em instância. Vide exemplo:
```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "echo The server's IP address is ${self.private_ip}"
  }
}
```
* Onde o ```local-exec``` executa o comando na máquina de origem, não na máquina de destino.
* Neste caso não é possível fazer referências a objetos externos a este recurso, então tem que-se utilizar o objeto self. Ele é usado junto com os atributos que são gerados na concepção do recurso, no caso do exemplo, o atrbuto private_ip.

## Creation-Time Provisioners
Por padrão, o provisioner é executado na hora que o recurso é criado. Ou seja, se o recurso é marcado para modificação, ou para ser destruído, o provisioner não vai rodar.  
Se a criação falhar, o recurso é marcado como **tainted**, que basicamente é uma marcação que o recurso recebe para ser destruído e recriado.

## Arguemntos padrões dos Provisioners
1. ```when```: Altera o creation time dos providers, e.g. when = "destroy".
```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    when    = "destroy"
    command = "echo 'Destroy-time provisioner'"
  }
}
```
**NOTE: A destroy-time provisioner within a resource that is tainted will not run. This includes resources that are marked tainted from a failed creation-time provisioner or tainted manually using terraform taint.** 
Logo, não adianta colocar o  when = "destroy", se o recurso estiver marcado como tainted.  
2. ```on_failure```

## Multiple provisioners
Multiple provisioners can be specified within a resource. Multiple provisioners are executed in the order they're defined in the configuration file.  
You may also mix and match creation and destruction provisioners. Only the provisioners that are valid for a given operation will be run. Those valid provisioners will be run in the order they're defined in the configuration file.  
```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "echo first"
  }

  provisioner "local-exec" {
    command = "echo second"
  }
}
``` 

## Failure Behavior
Por padrão em caso de falha, o recurso é marcado como tain para ser recriado depois, porém esse comportamento pode ser configurado.  
The ``` on_failure```  setting can be used to change this. The allowed values are:
* ``` "continue"```  - Ignore the error and continue with creation or destruction.
* ``` "fail"```  - Raise an error and stop applying (the default behavior). If this is a creation provisioner, taint the resource.

## remote-exec Provisioner
Ao contrário do local provisioner, o remote-exec executa o provisioner na máquina destino, onde se está executando o terraform.   
Existem três formas de passar os comandos:
* ```inline``` - This is a list of command strings. They are executed in the order they are provided. This cannot be provided with script or scripts.
* ``` script``` - This is a path (relative or absolute) to a local script that will be copied to the remote resource and then executed. This cannot be provided with inline or scripts.
* ```scripts ```- This is a list of paths (relative or absolute) to local scripts that will be copied to the remote resource and then executed. They are executed in the order they are provided. This cannot be provided with inline or script.  
The remote-exec provisioner supports both ssh and winrm type connections.   
Para usar o ssh é necessário configurar o Provisioner Connection, senão o remote-exec vai tentar logar via ssh com o usuário root sem senha.
https://www.terraform.io/docs/provisioners/connection.html  
```
# Copies the file as the root user using SSH
provisioner "file" {
  source      = "conf/myapp.conf"
  destination = "/etc/myapp.conf"

  connection {
    type     = "ssh"
    user     = "root"
    password = "${var.root_password}"
    host     = "${var.host}"
  }
}
```
Neste caso, o provisioner acontece no terraform apply. 
* O termo correto para configuração da máquina após ela ser criada, é o provisionamento.

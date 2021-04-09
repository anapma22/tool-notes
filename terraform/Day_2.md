____
# Módulos - Aula 1
A module is a container for multiple resources that are used together.  
https://www.terraform.io/docs/configuration/modules.html  
Forma como você pode reunir todas as suas configurações, recursos, datas, outputs em um lugar só.  
É possível configurar um módulo inteiro e mandar pra outro time. Ou seja, uma forma de contralizar os conhecimentos.  
Os módulos podem ser comparados as classes de desenvolvimento.  

## Root module
Every Terraform configuration has at least one module, known as its root module, which consists of the resources defined in the .tf files in the main working directory.  
Qualquer arquivo que você tenha e não definiu a configuração de módulo está dentro do root module. Esse é o caso de todos os arquivos que temos até agora.  
O output do module root da origem ao IP público.
* O ideal é que o main.tf (com o provider e terraform) sempre esteja module root.

## Child Module
To call a module means to include the contents of that module into the configuration with specific values for its input variables. Modules are called from within other modules using module blocks:
```
module "servers" {
  source = "./app-cluster"

  servers = 5
}
```
A module that includes a module block like this is the calling module of the child module.  
Pode ser comparado com um método.  
O output do child module volta **apenas** para o module root. A saída da combinação dos modules já citados instanciam resources que para os providers.
* Não é o módulo ideal.    
Foi criado terrafile.tf para ser o module child.
O output do module root é mostrado assim que damos o terraform apply plano, mas os outputs dos modules chils são encaminhados diretamente para o module root, não sendo mostrados pra gente.

## Accessing Module Output Values
The resources defined in a module are encapsulated, so the calling module cannot access their attributes directly. However, the child module can declare output values to selectively export certain values to be accessed by the calling module.  
Em nosso caso, foi inserido o output do child module no terrafile.tf, com uma alteração no value para ele retornar:
```
output "ip_address" {
  value = module.servers.ip_address
}
```
Esse ip_address veio do servers que foi colocado como input no module child, que por sua vez com os resources gerou o uma máquina no provider. E o module child recebeu como atributo depois da criação do ec2, o ip dessa máquina criada, que é o output do child module.  
O que será feito em seguida é pegar esse output e mandar para o root module, para podermos vê esse ip_address.

### Route53
É um serviço da AWS de DNSaaS.

## Providers within Modules
In a configuration with multiple modules, there are some special considerations for how resources are associated with provider configurations.  
While in principle provider blocks can appear in any module, it is recommended that they be placed only in the root module of a configuration, since this approach allows users to configure providers just once and re-use them across all descendent modules.  
O ideal é que os providers estejam no root module. Nesta seção temos a possibilidade que se tem de reutilizar um provider em um module child.  

### Implicit Provider Inheritance
For convenience in simple configurations, a child module automatically inherits default (un-aliased) provider configurations from its parent. This means that explicit provider blocks appear only in the root module, and downstream modules can simply declare resources for that provider and have them automatically associated with the root provider configurations.  
Foi a forma que utilizamos aqui.  

### Passing Providers Explicitly
When child modules each need a different configuration of a particular provider, or where the child module requires a different provider configuration than its parent, the providers argument within a module block can be used to define explicitly which provider configs are made available to the child module. For example:
```
# The default "aws" configuration is used for AWS resources in the root
# module where no explicit provider instance is selected.
provider "aws" {
  region = "us-west-1"
}

# A non-default, or "aliased" configuration is also defined for a different
# region.
provider "aws" {
  alias  = "usw2"
  region = "us-west-2"
}

# An example child module is instantiated with the _aliased_ configuration,
# so any AWS resources it defines will use the us-west-2 region.
module "example" {
  source    = "./example"
  providers = {
    aws = "aws.usw2"
  }
}
```

## Multiple Instances of a Module
O mesmo módulo pode ser usado várias vezes, em nosso caso no arquivo terrrafile.tf. Isto, desde que o nome que o módulo está sendo chamado seja diferente, bem como os inputs também sejam diferentes. Alguns inputs podem ter o mesmo nome, mas vai dar conflito caso a maioria seja igual.


_______
# Backends - Aula 2
A "backend" in Terraform determines how state is loaded and how an operation such as apply is executed. This abstraction enables non-local file state storage, remote execution, etc.  
By default, Terraform uses the "local" backend.  
Temos o backend remoto, que é melhor para trabalhar com time, operações remotas e manter dados sendíveis fora do disco.  
Estamos usando dentro do main.tf o backend "s3".  

## First Time Configuration
O terraform init vai gerar o terraform.tfstate, independente do backend;

## Partial Configuration
You do not need to specify every required argument in the backend configuration. Omitting certain arguments may be desirable to avoid storing secrets, such as access keys, within the main configuration. When some or all of the arguments are omitted, we call this a partial configuration.
### Interactively
Terraform will interactively ask you for the required values, unless interactive input is disabled. Terraform will not prompt for optional values.
### File
A configuration file may be specified via the init command line. To specify a file, use the -backend-config=PATH option when running terraform init. 
### Command-line key/value pairs
To specify a single key/value pair, use the -backend-config="KEY=VALUE" option when running terraform init.  
```
terraform init \
    -backend-config="address=demo.consul.io" \
    -backend-config="path=example_app/terraform_state" \
    -backend-config="scheme=https"
```

## Changing Configuration
You can change your backend configuration at any time. You can change both the configuration itself as well as the type of backend (for example from "consul" to "s3").  
Terraform will automatically detect any changes in your configuration and request a reinitialization. As part of the reinitialization process, Terraform will ask if you'd like to migrate your existing state to the new configuration. This allows you to easily switch from one backend to another.  

## State Storage
Backends determine where state is stored. For example, the local (default) backend stores state in a local JSON file on disk.   
When using a non-local backend, Terraform will not persist the state anywhere on disk except in the case of a non-recoverable error where writing the state to the backend failed.   

## Manual State Pull/Push
You can still manually retrieve the state from the remote state using the ```terraform state pull``` command. This will load your remote state and output it to stdout. You can choose to save that to a file or perform any other operations. e.g. ```terraform state pull >> new.tfstate```  
You can also manually write state with ```terraform state push```. **This is extremely dangerous and should be avoided if possible**. This will overwrite the remote state. This can be used to do manual fixups if necessary.  e.g. ```terraform state push new.tfstate```  
Se quiser mudar o state file com o push, precisa aumentar o serial, senão o terraform vai detectar a mudança no arquivo, mas vai reclamar que é o mesmo serial.  

## State Locking
If supported by your backend, Terraform will lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state.  
State locking happens automatically on all operations that could write state. You won't see any message that it is happening. If state locking fails, Terraform will not continue.   

## dynamondb
https://www.terraform.io/docs/providers/aws/r/dynamodb_table.html  
Banco de dados da aws com key/value.  
O daemondb foi criado por meio de um arquivo, o dynamondb.tf:
```
resource "aws_dynamodb_table" "dynamodb-terraform-state-lock" { # Tipo do recurso, seguido pelo nome da tabela
  name           = "terraform-state-lock-dynamo"
  hash_key       = "LockID"
  read_capacity  = 20
  write_capacity = 20
  
    attribute {
    name = "LockID"
    type = "S"
  }
    tags = {
    Name        = "dynamodb-table-1"
  }
}
```
Agora, vendo a documentação do S3, é necessário apenas dizer qual é a tabela do dynamondb que será utilizada.   
Essa linha foi inserida dentro do main.tf, dentro do bloco terraform: ```dynamodb_table = "terraform-state-lock-dynamo"```.
A partir de agora, está sendo dito ao backend S3 para oferecer a funcionalidade do state lock.  
Ao tentar usar o state ao mesmo tempo é mostrada a seguinte mensagem de erro:
```
Error: Error locking state: Error acquiring the state lock: ConditionalCheckFailedException: The conditional request failed
        status code: 400, request id: 1QQEDQ17L8AJB9FI8BKO64I7M7VV4KQNSO5AEMVJF66Q9ASUAAJG
Lock Info:
  ID:        tururu
  Path:      tururu/terraform-test.tfstate
  Operation: OperationTypePlan
  Who:       
  Version:   0.12.24
  Created:   2020-04-29 02:59:37.600677576 +0000 UTC
  Info:      


Terraform acquires a state lock to protect the state from being written
by multiple users at the same time. Please resolve the issue above and try
again. For most commands, you can disable locking with the "-lock=false"
flag, but this is not recommended.
```

#### Possível problema
Na hora de destruir, o terraform destroy vai apagar também a tabela do dynamondb. Então, pode ser que o terraform retorne um erro no final, porque ele vai perder o state lock, já que a tabela do dynamondb já foi apagada.  
Possíveis soluções:
1. Comentar a linha ```dynamodb_table = "terraform-state-lock-dynamo"``` dentro do main.tf.
2. Passar o comando: ```terraform destroy -lock=false```. Para voltar a ter o lock state, é necessário passsar ```terraform plan -out plan -lock=false```, porque a tabela do dynamondb não existe ainda.

______
# Introdução ao state - Aula 3
Forma do terraform guardar as informações, por padrão é colocado no terraform.tfstate caso você não coloque nada.  
O terraform não funciona sem state, ele é necessário para mapear o mundo real (cloud).

## Mapping to the Real World
Terraform requires some sort of database to map Terraform config to the real world. When you have a resource ```resource "aws_instance" "foo"``` in your configuration, Terraform uses this map to know that instance ```i-abcd1234``` is represented by that resource.  
Ou seja, é o mapeamento dos bancos de definições para a cloud.  


## Metadata
Ajuda o terraform a entender quais recursos são dependentes de outros, e.g. quando se paga um recurso fornence dependência a outro, ambos os recursos são apagados de uma vez.   
Da mesma forma, também se evita de apagar um recurso que está sendo requerido pelo outro.   
Terraform typically uses the configuration to determine dependency order. However, when you delete a resource from a Terraform configuration, Terraform must know how to delete that resource. Terraform can see that a mapping exists for a resource not in your configuration and plan to destroy. However, since the configuration no longer exists, the order cannot be determined from the configuration alone.  


## Performance
Já tem uma espácie de cache do que se tem dentro da cloud.  
In addition to basic mapping, Terraform stores a cache of the attribute values for all resources in the state. This is the most optional feature of Terraform state and is done only as a performance improvement.  
When running a terraform plan, Terraform must know the current state of resources in order to effectively determine the changes that it needs to make to reach your desired configuration.  
**For small infrastructures, Terraform can query your providers and sync the latest attributes from all your resources. This is the default behavior of Terraform: for every plan and apply, Terraform will sync all resources in your state.**  
Ou seja, quando você tem uma infraestrtutura pequena, o terraform pergunta ao provider se houve alguma mudança em relação aos recursos que estão la. Se sim, o terraform atualiza o state antes de rodar o plano e antes de rodar o apply. Isso é bom porque o state está atualizado, você consegue fazer avaliações apenas com base neste arquivo.  
Lembrando que esse comportamento é o padrão, que utiliza o refresh.  
**For larger infrastructures, querying every resource is too slow.**  
Caso em grandes infraestruturas queira desativar o recurso do refresh, é necessário fazer uso das flags: ```-refresh=false```. 

## Syncing
In the default configuration, Terraform stores the state in a file in the current working directory where Terraform was run. This is okay for getting started, but when using Terraform in a team it is important for everyone to be working with the same state so that operations will be applied to the same remote objects.  
Remote state is the recommended solution to this problem.   
A ideia do syncing é para que todo mundo do time tenha uma foto da sua infra de forma atualizada sempre, independente da flag refresh esteja como true ou false.  

## Refresh
Por padrão todo plan e apply utiliza o refresh, mas não é necessário rodar um plan para utilizar o refresh, existe o comando separado.  
Para utilizar esse comando, toda a infra já está up. Em seguida, usamos o ```terraform state pull > stateaula.tfstate```, que vai no state remote e baixa o state para o arquivo stateaula.tfstate.  
Agora, temos um json com o state atual.  
Como exemplo, no console da aws foi alterado o nome da tag da instância ec2. 
Após isso é dado o comando ```terraform refresh```. Agora, o terraform vai na AWS e pergunta se houve alguma mudança e em seguida atualiza o state.  
Para vê o arquivo do state atualizado, damos novamente o ```terraform state pull > stateaula.tfstate```. Já recebemos o state atualizado.  
Ou seja, aqui o que foi alterado manualmente na AWS é atualizado para o state por meio do refresh. Ao da o apply a state será colocado na AWS.  

## Manipular state
O tfstate é um arquivo em json, e como tal pode ser manipulado por programas que tenham essa finalidade.  
Para instalar no container um binário para manipular arquivos do tipo json: ```apk -U add jq```.  
Para usar o binário com o state, fazendo uma busca a partir do resource: ```terraform state pull | jq .resources```, e o resources será mostrado na tela.

____
# State avançado  - Aula 4
Como já mostrado anteriormente, **não é aconselhavel alterar de forma manual o statefile**, existe um comando específico para manipulação deste arquivo.  
The terraform state command is used for advanced state management. As your Terraform usage becomes more advanced, there are some cases where you may need to modify the Terraform state. Rather than modify the state directly, the terraform state commands can be used in many cases instead.  

## Usage
```terraform state <subcommand> [options] [args]```
* ```list```: Lista todos os recursos do terraform. Pode ser filtrado por instâncias, módulo, ids...
* ```mv```: Pode renomear um recurso do state file ou criar um child módule.
### Rename a Resource
The example below renames the packet_device resource named worker to helper:
 ```terraform state mv 'packet_device.worker' 'packet_device.helper```  
Se apenas modificar o state e não modificar os arquivos, no final valerá o que está no arquivo, ou seja, o state atual sem nenhuma modificação.
### Move a Resource Into a Module
Após a criação de um child module, é possível mover um recurso do root module para o novo child module.  
The example below moves the packet_device resource named worker into a module named app. The module will be created if it doesn't exist.
``` terraform state mv 'packet_device.worker' 'module.app'```  

* ```pull```: Baixa o state file.
* ```push```: Pega um arquivo local e manda state file.
* ```rm```: Não remove os recursos do seu provider, apenas os tira do state file. Se der o terraform plan, será mostrado a criação do recurso que foi excluído com o rm, para aplicar as alterações do rm, preciso da o terraform apply logo após o terraform state rm.  
**Caso apague apenas do state file, o recurso continará a existir, mas fora do controle do terraform! Cuidado!**
Items removed from the Terraform state are not physically destroyed. Items removed from the Terraform state are only no longer managed by Terraform. For example, if you remove an AWS instance from the state, the AWS instance will continue running, but terraform plan will no longer see that instance.
* ```show```:

## Backups
Todas as alterações no state file geram um backup automático.  
All terraform state subcommands that modify the state write backup files. Note that backups for state modification can not be disabled. Due to the sensitivity of the state file, Terraform forces every state modification command to write a backup file.  

## Command-Line Friendly
The output and command-line structure of the state subcommands is designed to be easy to use with Unix command-line tools such as grep, awk, etc.  

____
# Importando recursos - Aula 5

O terrarform importa é responsável por trazer um infraestrutura legada para a cloud. Esse comando não gera códigos, apenas altera o state.

## Import
Terraform is able to import existing infrastructure. This allows you take resources you've created by some other means and bring it under Terraform management.  
This is a great way to slowly transition infrastructure to Terraform, or to be able to be confident that you can use Terraform in the future if it potentially doesn't support every feature you need today.    
To import a resource, first write a resource block for it in your configuration, establishing the name by which it will be known to Terraform:
```
resource "aws_instance" "example" {
  # ...instance configuration...
}
```
The name "example" here is local to the module where it is declared and is chosen by the configuration author. This is distinct from any ID issued by the remote system, which may change over time while the resource name remains constant.  
If desired, you can leave the body of the resource block blank for now and return to fill it in once the instance is imported.  
Now terraform import can be run to attach an existing instance to this resource configuration:
```terraform import aws_instance.example i-abcd1234```  
This command locates the AWS instance with ID i-abcd1234.  
Para saber o ID da instancia, isso é presente na documentação de cada tipo de recurso.
Para o AWS instance, temos: https://www.terraform.io/docs/providers/aws/r/instance.html, no final tem a seção Import. Por padrão, para que o import funcione, é necessário ter o resource no root module. Ainda é possível ainda fazer o import de um child module, vide a documentação para isso.

## Complex Imports
The above import is considered a "simple import": one resource is imported into the state file. An import may also result in a "complex import" where multiple resources are imported.   
For example, an AWS security group imports an aws_security_group but also one aws_security_group_rule for each rule.  
In this scenario, the secondary resources will not already exist in configuration, so it is necessary to consult the import output and create a resource block in configuration for each secondary resource. If this is not done, Terraform will plan to destroy the imported objects on the next run.  


_____
# Workspace - Aula 6
É uma configuração feita no state. É uma possiblidade de apontar para o mesmo state file, tendo a mesma configuração do backend também, mas usando múltiplos states files para gerenciar seus states. Ou seja, uma forma de usar uma única configuração, mas com múltiplos modelos, múltiplas instâncias.  
Each Terraform configuration has an associated backend that defines how operations are executed and where persistent data such as the Terraform state are stored.  
The persistent data stored in the backend belongs to a workspace. Initially the backend has only one workspace, called "default", and thus there is only one Terraform state associated with that configuration.  
Certain backends support multiple named workspaces, allowing multiple states to be associated with a single configuration. The configuration still has only one backend, but multiple distinct instances of that configuration to be deployed without configuring a new backend or changing authentication credentials.  
## Using Workspaces
Workspaces are managed with the ```terraform workspace``` set of commands. To create a new workspace and switch to it, you can use ```terraform workspace new```; to switch workspaces you can use ```terraform workspace select```; etc.

**Workspace trabalha apenas a nível de state, não fazendo nenhuma segmentação na cloud, no SaaS.** e.g. foi criado dois workerspaces w1 e w2, em um w1 foi inserido um ec2, em seguida foi adicionado um ec2 em w2, ambos ec2s estarão no mesmo lugar. Logo, vai haver conflitos, pois estão tudo no mesmo lugar na cloud, a segmentação é apenas a nível de state, que impacta no terraform plan.  
Ao criar um novo workspace: ```terraform workspace new production```, automaticamente o workspace é trocado para o production.
## Comandos
1. ```new```: Cria um workspace. ```terraform workspace new name_workspace```
2. ```list```: Lista todos os workspaces. ```terraform workspace list```
3. ```show```: Mostra seu workspace atual.
4. ```select```: Troca de workspace.
5. ```delete```: Deleta um workspace.  
Não é possível apagar o seu workspace atual.  
Caso você tente apagar um workspace que tem resources o terraform avisa que o workspace não está vazio. Porém resulta neste warning:
```
WARNING: "staging" was non-empty.
The resources managed by the deleted workspace may still exist,
but are no longer manageable by Terraform since the state has
been deleted.
```

É comum usar o workspace como enviroment, e.g. production, staging.

## Current Workspace Interpolation
Permite que os workspaces (e.g. prodcution e staging) fiquem isolados por regiões, cada um é criado numa região diferente. Assim os recursos de um workspace não conflitam com o do outro.    
Referencing the current workspace is useful for changing behavior based on the workspace. For example, for non-default workspaces, it may be useful to spin up smaller cluster sizes. For example:
```
resource "aws_instance" "example" {
  region = "${terraform.workspace == "default" ? 5 : 1}" 

  # ... other arguments
}
```
* 5 é o que é executado se a verificação for true e 1 é se for false.
  * region pode ser trocado por outro argumento.  

______
# Dados sensíveis dentro do state - Aula 7
São basicamente dados que você não quer expor para a internet. É necessário mante-los seguros.  
Tudo que é escrito na cloud, também vai para o state, então esse arquivo vai conter senhas, acessos que irão para o json file.  
Como prática de segurança tem o uso de backend remoto e não fazer commit no github do statefile.  
Mesmo o backend sendo remoto, os dados não estão criptografados. Mas isso pode ser resolvido facilmente: The S3 backend supports encryption at rest when the encrypt option is enabled. IAM policies and logging can be used to identify any invalid access. Requests for the state go over a TLS connection.  
Basta adicionar este linha no backend: ```encrypt = true``` e em seguida destruir e recriar o backend.  
Para outros backends é necessário checar a documentação se eles aceitam.  




______
# Bloco Dinâmico - Aula 1
## Blocos
https://www.terraform.io/docs/providers/aws/r/instance.html  (ebs_block_device)
Permite que sejam criados discos na AWS. Foi inserido o código abaixo dentro de resources:
```
  ebs_block_device {
    device_name           = "/dev/sgd"
    volume_size           = 5
    volume_type           = "gp2" #influencia na forma de cobrança
  }
```
Para usar mais de um disco, não é viável copiar e colar o código acima. O for_each não funcionaria pois ele pegaria o recurso inteiro, e o que precisamos é apenas de um for neste bloco específico, ai que entra o bloco dinâmico.  

## dynamic blocks
https://www.terraform.io/docs/configuration/expressions.html#dynamic-blocks  
Permite que se pegue um bloco aninhado, um bloco que esta dentro do bloco inicial. O objetivo é iterar apenas o bloco aninhado.  
You can dynamically construct repeatable nested blocks like ```setting``` using a special ```dynamic``` block type, which is supported inside ```resource```, ```data```, ```provider```, and ```provisioner``` blocks:
```
resource "aws_elastic_beanstalk_environment" "tfenvtest" {
  name                = "tf-test-name"
  application         = "${aws_elastic_beanstalk_application.tftest.name}"
  solution_stack_name = "64bit Amazon Linux 2018.03 v2.11.4 running Go 1.12.6"

  dynamic "setting" {
    for_each = var.settings
    content {
      namespace = setting.value["namespace"]
      name = setting.value["name"]
      value = setting.value["value"]
    }
  }
}
```
No exemplo acima, o ```for_each``` não vai criar um recurso inteiro, porque ele esta dentro do bloco ```dynamic "setting" ```, apenas o bloco será iterado e executado como um laço de repetição.  
Se o ```var.settings``` for uma lista com três strings, o laço será executado três vezes.   
**Ou seja o ```for_each``` vai criar apenas um recurso com três discos, os ```ebs_block_device```.**
O código para o ebs_block_device foi alterado da seuinte forma dentro do ec2.tf:
```
  dynamic "ebs_block_device" {
    for_each = var.blocks
    content {
      device_name = ebs_block_device.value["device_name"]
      volume_size = ebs_block_device.value["volume_size"]
      volume_type = ebs_block_device.value["volume_type"]
    }
  }
```
Sendo necessário a criação da variável blocks, para isso foi criado um arquivo ```terraform.tfvars``` que é carregado automaticamente. Blocks é uma lista de objetos.
```
  blocks = [ #lista
  { #primeiro objeto
    device_name           = "/dev/sgd"
    volume_size           = 5
    volume_type           = "gp2" #influencia na forma de cobrança
  },
  { #segundo objeto
    device_name           = "/dev/sdh"
    volume_size           = 10
    volume_type           = "gp2"
  }
  ]
```
No arquivo ```variable.tf``` será usado o que foi aprendido no type constraints, que é basicamente indicar qual tipo determinada variável deve ter:
```
variable "blocks" {
  type = list(object({
    device_name = string
    volume_size = string
    volume_type = string
  }))
  description = "List of EBS block"
}
```
Após essas três configurações é possível vê no terraform plan, a cria,ão dos três diferentes discos dentro do mesmo recurso.

Outro uso é nos módulos: Ao criar um módulo, pode-se adicionar um feature que é criar instâncias, o ideal é que possa ser passado via variável quantos devices e qual configuração você quer.  
Assim é possível criar uma variável sobre quem esta usando o módulo e não no módulo em si, e.g. uma pessoa pediu a criação de três discos, mas você não precisa forçar todo seu time a usar esses discos, se apenas uma pessoa é que precisa disso.
______
# String Template
É a ideia de pegar a string e tornar seu valor em dinâmico.  
https://www.terraform.io/docs/configuration/expressions.html#string-templates  
The sequences ```${``` and ```%{``` begin template sequences. Templates let you directly embed expressions into a string literal, to dynamically construct strings from other values.  

## Interpolation
A ```${ ... }``` sequence is an interpolation, which evaluates the expression given between the markers, converts the result to a string if necessary, and then inserts it into the final string:
```"Hello, ${var.name}!"```  
In the above example, the named object ```var.name``` is accessed and its value inserted into the string, producing a result like "Hello, Juan!".

## Directives
A ```%{ ... }``` sequence is a directive, which allows for conditional results and iteration over collections, similar to conditional and ```for``` expressions.
* The ```if <BOOL>/else/endif``` directive chooses between two templates based on the value of a bool expression: 
```
"Hello, %{ if var.name != "" }${var.name}%{ else }unnamed%{ endif }!"
```

Ou seja, a interpolação permite trazer váriaveis de uma forma que não seja hardcode e as diretivas permitem trabalhar com condições na escolha das variáveis.

______
# Terraform Cloud
https://www.terraform.io/docs/cloud/index.html  
 
## About Terraform Cloud and Terraform Enterprise
Terraform Cloud is an application that helps teams use Terraform together. It manages Terraform runs in a consistent and reliable environment, and includes easy access to shared state and secret data, access controls for approving changes to infrastructure, a private registry for sharing Terraform modules, detailed policy controls for governing the contents of Terraform configurations, and more.  
Terraform Cloud is available as a hosted service at https://app.terraform.io. We offer free accounts for small teams, and paid plans with additional feature sets for medium-sized businesses.  
Large enterprises can purchase Terraform Enterprise, our self-hosted distribution of Terraform Cloud. It offers enterprises a private instance of the Terraform Cloud application, with no resource limits and with additional enterprise-grade architectural features like audit logging and SAML single sign-on.  

## Workspaces
Tudo faz parte do worspace no terraform cloud. É um SaaS do HasiCorp.    
Faz muitas abstrações, como state, variáveis...  
Workspaces are how Terraform Cloud organizes infrastructure.  
**Terraform Cloud and Terraform CLI both have features called "workspaces," but they're slightly different. CLI workspaces are alternate state files in the same working directory; they're a convenience feature for using one configuration to manage multiple similar groups of resources.**

### Workspace Contents
* Terraform configuration: In linked version control repository, or periodically uploaded via API/CLI.  
* Variable values and State: In workspace.  
* Credentials and secrets: In workspace, stored as sensitive variables.  
Acessa e cria uma conta free: https://app.terraform.io/session  
Autoriza o github.com e seleciona um repositório. Após isso sera criado o workspace, é necessário configurar as variáveis.  
These variables are used for all plans and applies in this workspace. Workspaces using Terraform 0.10.0 or later can also load default values from any *.auto.tfvars files in the configuration.  
Sensitive variables are hidden from view in the UI and API, and can't be edited.   
## Terraform Variables
These Terraform variables are set using a terraform.tfvars file.
## Environment Variables
These variables are set in Terraform's shell environment using export.  
Após isso na baa superior "Runs" e necessário rodar manualmente o primeiro plan.  
para uso no github, é recomendável após a modificação criar uma branch e commitar nesta branch, é indicado que a mensagem seja o motivo.  
Após isso se faz um pull request, é indicadoq ue aqui seja colocado como valdiar o PR. No nosso caso, seria como testar via terraform cloud.    

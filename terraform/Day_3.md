# Conseguindo ajuda - Aula 1
https://www.terraform.io/docs/commands/index.html 
To get help for any specific command, pass the -h flag to the relevant subcommand. For example, to see help about the graph subcommand: ```terraform graph -h```.  

## Shell Tab-completion
If you use either bash or zsh as your command shell, Terraform can provide tab-completion support for all command names and (at this time) some command arguments.  
To add the necessary commands to your shell profile, run the following command: ```terraform -install-autocomplete```.   
After installation, it is necessary to restart your shell or to re-read its profile script before completion will be activated: ```bash```, em seguida ```exit```.  


_________
# Dependências entre recursos -  Aula 2
Dependência é quando um recurso, na hora do terraform apply, deve saber qual recurso ele deve rodar primeiro, em paralalelo ou em sequência.  
https://learn.hashicorp.com/terraform/getting-started/dependencies  

## Implicit and Explicit Dependencies
By studying the resource attributes used in interpolation expressions, Terraform can automatically infer when one resource depends on another.  
Terraform uses this dependency information to determine the correct order in which to create the different resources. 

### Implicit Dependencies
Implicit dependencies via interpolation expressions are the primary way to inform Terraform about these relationships, and should be used whenever possible.  
O terraform ja entende, esse deve ser o preferível.  
Para facilitar teremos dois recursos, A e B, onde B depende de A. A é criado inicialmente e B sera depois, não faz sentido ser criado logo B, pois ele depende de outro recurso, A, que não existe.  Da mesma forma, na hora do terraform destroy, é destruído logo o recurso dependente, B, para depois ser destruido A.

### Explicit Dependencies
Sometimes there are dependencies between resources that are not visible to Terraform. The ```depends_on``` argument is accepted by any resource and accepts a list of resources to create explicit dependencies for.  
Por exemplo, para criar duas intâncias EC2, onde a segunda só deve existir após a primeira, o depends_on é dentro do resource.  
Abaixo tem um exemplo:
```
resource "aws_instance" "web2" {
  count         = var.num_servers
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"

  tags = {
    Name = "HelloWorld"
  }
  depends_on = [aws_instance.web]
}
```

_______
# Introdução a comandos e mudando o verbose deles - Aula 3

https://www.terraform.io/docs/internals/debugging.html  


Terraform has detailed logs which can be enabled by setting the TF_LOG environment variable to any value. This will cause detailed logs to appear on stderr.  
You can set TF_LOG to one of the log levels TRACE, DEBUG, INFO, WARN or ERROR to change the verbosity of the logs. TRACE is the most verbose and it is the default if TF_LOG is set to something other than a log level name.  
To persist logged output you can set TF_LOG_PATH in order to force the log to always be appended to a specific file when logging is enabled.  
O comando é usado da seguinte forma: ```TF_LOG=DEBUG command```

___________
# Comando taint - Aula 4
## terraform state list
Serve para listar as datas e resources, ou seja, lÊ o que está no state e mostra essas informações.  

## taint
https://www.terraform.io/docs/commands/taint.html  
The ```terraform taint``` command manually marks a Terraform-managed resource as tainted, forcing it to be destroyed and recreated on the next apply.  
Exemplo de motivo para recriar uterm resource: Ter um ec2 ou qualquer outra coisa de forma equivocada.
### Usage
```terraform taint [options] address```  
e.g.: ```terraform taint aws_instance.web[0]```.

## untaint
Para desfazer o taint é usado o:```terraform untaint aws_instance.web[0]```.  

_______
# Introdução a grafos - Aula 5
https://www.youtube.com/watch?v=Frmwdter-vQ&feature=youtu.be
Independente do problema, o objetivo é respresentar o mesmo por elementos simplificados que permitam uma melhor visualização.  
Vértice: representação de um elemento. Levam ao conjunto de vértices (V).
Aresta: ligação entre dois vértices. Levam ao conjunto de elos (E).  


_______
# Comando graph - Aula 6
## Command: graph
The terraform graph command is used to generate a visual representation of either a configuration or execution plan. The output is in the DOT format, which can be used by GraphViz to generate charts.
### Usage
```terraform graph [options] [DIR]```   
1. ```draw-cycles``` - Highlight any cycles in the graph with colored edges. This helps when diagnosing cycle errors.
2. ```type=plan``` - Type of graph to output. Can be: plan, plan-destroy, apply, validate, input, refresh.
3. ```module-depth=n``` - (deprecated) In prior versions of Terraform, specified the depth of modules to show in the output.
É necessário instalar o graphviz para visualizar o arquivo .dot: ```apk add -U graphviz```.  
### Generating Images
Será gerado um gráfico do grafo de nossa infraestrura com o teerraform: ```terraform graph | dot -Tsvg > graph.svg```.  

_____
# Comando fmt - Aula 7
O comando ```terraform format``` vem da palavra format. 
The terraform fmt command is used to rewrite Terraform configuration files to a canonical format and style.  

## Style Conventions
Note: This page is about Terraform 0.12 and later.  
https://www.terraform.io/docs/configuration/style.html  

## Usage
```terraform fmt [options] [DIR]```
1. ```list=false``` - Don't list the files containing formatting inconsistencies.
2. ```write=false``` - Don't overwrite the input files. (This is implied by -check or when the input is STDIN.)
3. ```diff``` - Display diffs of formatting changes.  
Para corrigir e saber o que foi alterado: ```terraform fmt  -diff```.  
O que tiver - na frente indica o que está errado e o que tiver + é o correto.
4. ```check``` - Check if the input is formatted. Exit status will be 0 if all input is properly formatted and non-zero otherwise.  
A saída de todo comando linux retorna 0 se foi executado corretamente e 1 se ocorreu algum erro.  
Todo pipeline quebra caso a saída padrão seja diferente de 0, é possível usar o fmt check para verificar isso. Caso esteja errado, ele vai avisar qual o arquivo contém o erro.   
5. ```recursive``` - Also process files in subdirectories. By default, only the given directory (or current directory) is processed.

__________
# Comando validate - Aula 8
Muito importante para pipeline, pois é pegando a ideia básica do CI é que as coisas que tenham feedback mais rápido, venham primeiro. Este é o caso do validade, que tem execução rápida. Ou seja, se no pipline já tiver terraform, o validade deve vir no começo.  
Ele valida se o código tem algum erro sintático. Frisando que esse comando não vai no provider ou no state remoto, ele fica apenas no código.  
The terraform validate command validates the configuration files in a directory, referring only to the configuration and not accessing any remote services such as remote state, provider APIs, etc.  
Validation requires an initialized working directory with any referenced plugins and modules installed. To initialize a working directory for validation without accessing any configured remote backend, use:```terraform init -backend=false```.  
## Usage
```terraform validate [options] [dir]```
The command-line flags are all optional. The available flags are:
1. ```json``` - Produce output in a machine-readable JSON format, suitable for use in text editor integrations and other automated systems. Always disables color.
2. ```no-color``` - If specified, output won't contain any color.  
Caso encontre algum erro, ele mostrará qual o recurso, o arquivo, a linha o que está faltando.  





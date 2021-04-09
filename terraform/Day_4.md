# Condições - Aula 1
https://www.terraform.io/docs/configuration/expressions.html#arithmetic-and-logical-operators  
## Arithmetic and Logical Operators
An operator is a type of expression that transforms or combines one or more other expressions. Operators either combine two values in some way to produce a third result value, or transform a single given value to produce a single result.  
When multiple operators are used together in an expression, they are evaluated in the following order of operations:
1. ```!, - (multiplication by -1)```
2. ```*, /, %```
3. ``` +, - (subtraction)```
4. ```>, >=, <, <=```
5. ```==, !=```
6. ```&&```
7. ```||```

## Conditional Expressions
A conditional expression uses the value of a bool expression to select one of two values.
The syntax of a conditional expression is as follows: ```condition ? true_val : false_val```.  
If ```condition``` is true then the result is ```true_val```. If condition is false then the result is ```false_val```.  
e.g.: ```count = var.workspace == "production" ? 2 : 1```.  
### count.index
Pega o índice, o endereço único de cada resourse. e.g:``` instance_type = count.index < 1 ? "t2.micro" : "t3.medium"```.  


______
# Type constraints - Aula 2
Coisas importantes do módulo: input e output. Serão tratadas restrições do input.
É importante usar os types constrainsts, porque assim você garante que input seja exatamente do mesmo type que você espera, o que reduz sua chance de erros.
O ideal não é que a pessoa fique entrando no módulo para entender sua lógica e sim que entenda os inputs e outpts e consequentemente os types de cada um deles.   
https://www.terraform.io/docs/configuration/types.html  
Terraform module authors and provider developers can use detailed type constraints to validate user-provided values for their input variables and resource arguments.  
## Primitive Types
* ```string```: a sequence of Unicode characters representing some text, such as "hello".
* ```number```: a numeric value. The number type can represent both whole numbers like 15 and fractional values such as 6.283185.
* ```bool```: either true or false. bool values can be used in conditional logic.  
### Conversion of Primitive Types
* ```true``` converts to "true", and vice-versa --> bool to string
* ```false``` converts to "false", and vice-versa --> bool to string
* ```15``` converts to "15", and vice-versa --> number to string

## Complex Types
A complex type is a type that groups multiple values into a single value. Complex types are represented by type constructors, but several of them also have shorthand keyword versions.
## Collection Types
A collection type allows multiple values of one other type to be grouped together as a single value.  
For example, the type list(string) means "list of strings", which is a different type than list(number), a list of numbers. All elements of a collection must always be of the same type.  
The three kinds of collection type in the Terraform language are:
* list(...): a sequence of values identified by consecutive whole numbers starting with zero.  
The keyword list is a shorthand for list(any), which accepts any element type as long as every element is the same type. 
* map(...): a collection of values where each is identified by a string label.
The keyword map is a shorthand for map(any), which accepts any element type as long as every element is the same type.
* set(...): a collection of unique values that do not have any secondary identifiers or ordering.  

## Structural Types
A structural type allows multiple values of several distinct types to be grouped together as a single value. Structural types require a schema as an argument, to specify which types are allowed for which elements.  
The two kinds of structural type in the Terraform language are: 
* object(...): a collection of named attributes that each have their own type.  
The schema for object types is ```{ <KEY> = <TYPE>, <KEY> = <TYPE>, ... }``` — a pair of curly braces containing a comma-separated series of ```<KEY> = <TYPE> ```pairs. 
* tuple(...): a sequence of elements identified by consecutive whole numbers starting with zero, where each element has its own type.
The schema for tuple types is ```[<TYPE>, <TYPE>, ...] ```— a pair of square brackets containing a comma-separated series of types.  
 
### Resource: aws_instance
https://www.terraform.io/docs/providers/aws/r/instance.html  
#### Argument Reference
```vpc_security_group_ids``` - (Optional, VPC only) A list of security group IDs to associate with.   
Ao passar um ```export TF_VAR_securitygroup=[22,10,33]```, o terraform validate e terraform plan são executados corretamente, mesmo essa lista não tendo nenhuma relação com o VPC. Isso nos mostra que alguns testes são feitos no ```terraform plan``` e outros são feitos apenas no ```terraform apply```. Nos casos onde o problema ocorre no terraform apply é ruim pois serão gerados alguns recrusos caso estejam corretos e os que tiver erros não serão gerados, ou seja, vai subir um ambiente bagunçado e incompleto.  
O terraform não tem roolback e ai está demonstrado a importância de ter um ambiente de teste.  

______
# Laço for_each - Aula 3
https://www.terraform.io/docs/configuration/resources.html#for_each-multiple-resource-instances-defined-by-a-map-or-set-of-strings

Tem o ```for``` e o ```for_each```, onde este último é usado dentro do recurso. Ele é usado para passar variáveis diferentes a cada execução do loop.  
A given resource block cannot use both count and for_each.  
The for_each meta-argument accepts a map or a set of strings, and creates an instance for each item in that map or set. Each instance has a distinct infrastructure object associated with it, and each is separately created, updated, or destroyed when the configuration is applied.  
Este exemplo foi inserido no ec2.tf:  
```
  for_each = {
    dev    = "t2.micro"
    stagig = "t3.medium"
  }
```
Mas retornou o erro:
```
Error: Unsupported attribute

  on output.tf line 2, in output "ip_address":
   2:   value = "${aws_instance.web[*].public_ip}"

This object does not have an attribute named "public_ip".
```
Isso aconteceu porque o output estava assim:
```
output "ip_address" {
  value = "${aws_instance.web[*].public_ip}"
}
```
Sendo exibido cada IP público para cada indice e tudo estava sendo jogado no value, mas agora estamos usando o for each, que não tem um public dentro de algum indice. 
### Using Sets
The Terraform language doesn't have a literal syntax for sets, but you can use the toset function to convert a list of strings to a set:
```
for_each =  toset(var.instance_type) #substituiu o bloco do for_each anterior em ec2.tf
```
Porém o erro ainda persiste, pois o jeito de se acessar os valores são diferentes no for_each, não existindo indices, como está sendo requisitado no ```output.tf```. Para isso é necessário alterar este arquivo da seguinte maneira:
```
output "ip_address" {
  value = {
    for instance in aws_instance.web:
    instance.id => instance.private_ip
  }
}
```
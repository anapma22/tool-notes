# Aula ao vivo

## AMI
* EC2: Máquina virtual. Como um objeto.
* AMI: Equivalente ao snapshot, é possível subir outra instância a partir de uma AMI salva. COmo uma classe.  
O AWS também tem o conceito de snapshot, que é a foto em determinado momento.   
A AMI normalmente é feita para criar sua primeira máquina EC2, já que ela precisa da AMI. Como uma imagem padrão que habilita a criação do EC2. AMI é como a imagem molde para as máquinas do EC2.  
Existem AMI próprias da AWS e de outras fontes, cada caso deve ser analisado para escolher de onde a AMI deverá ser pega. e.g. Será criada uma EC2 do Ubuntu, então o ideal é que a fonte da AMI seja a canonical.  

## Data e resource
* Resource é criado pelo AWS, um recurso novo que até então não existia.
* Data não cria nada, ele colhe informações já existentes de um recurso previamente criado, seja por você ou por outra pessoa. No nosso caso, foi a usamos a imagem pública que o Gomex criou.
```
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["tururu-${var.hash_commit}"]
  }

  owners = ["tururu"] #mudar o filtro também
}
```

### Como buscar AMIs?
https://cloud-images.ubuntu.com/locator/ec2/ Aqui é possível ter uma ideia sobre a versão que se esta usando ou até mesmo procurar por uma AMI, porém não é aconselhavel usar hardcode, porque ao sofrer alguma atualziação, a AMI pode mudar.

### Automatizar AMI
Pega uma AMI pública, e vai em https://console.aws.amazon.com/ec2 em seguida em images e AMI, muda o filtro para public images. Seleciona a tag AMI ID e cola o AMI que foi copiado no início. 
Como resultado da pesquisa vai aparecer um AMI Name, esse campo deverá ser copiado até o último traço. e.g. AMI Name: ubuntu/images/hvm-ssd/ubuntu-focal-20.04-arm64-server-20200423, o que deve ser copiado é ubuntu/images/hvm-ssd/ubuntu-focal-20.04-arm64-server-. Os números são os que mudam após as atualizações.  
O que também deve ser alterado é o Owner, porque só assim para garantir que a AMI é de quem realmente você esta tentando pegar.
```
data "aws_ami" "ubuntu" { # o "ubuntu" é um nome interno, não tem nada com AWS e pode ser outro.
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-arm64-server-*"]
  }          # o * pega qualquer coisa que vinher depois do -

  owners = ["tururu"] 
}
```
Após isso é necessário criar outro bucket no S3 e atualizar o novo nome.
* Os AMI IDs diferem por regiões. Enquanto os AMI Names não se alteram em diferentes regiões, caso exista o AMI nas regioes em questao.  
A regiao é definida no provier.  

Em caso de problemas desconhecidos: **rm -fr .terraform/**

## S3
Como uma pasta que guarda arquivos na AWS.

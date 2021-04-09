# Aula 1

```ansible -i hosts all -m ping``` ou ```ansible -i hosts nome_maquina_ou_grupo_no_hosts -m ping```.

O parâmetro -k no comando acima pede a senha SSH com o usuário atual. Para logar com um user específico é adicionado o -u user no comando acima.

Para parar de aparecer warning: ```sudo sed -i 's\/#deprecation_warnings = True /deprecation_warnings = False/g' /etc/ansible/ansible.cfg```.

Para passar um comando específo: -a "command", ex: -a "bash -c 'uptime'" ou -m shell -a "uptime".

Para clonar um repositório para uma pasta específica: ```ansible -i hosts all -m git -a "repo=end dest=/local version=HEAD"```.

O módulo setup recupera os artifacts dos hosts.

Para usar o apt ou yum:ansible -i hosts -b -m apt -a "name-vim state=present". A flag -b indica o uso do sudo

Copiar a chave ssh para a máquina que será o server ansible, em seguida:
ssh-agent bash
ssh-add endereço_chave

Para liberar qualquer comando com um usuário não root
vim /etc/sudoers
user ALL=(ALL) NOPASSWD:ALL

# AWS
Foram preparadas três máquinas, uma que vai ser o server ansible e duas para receber as configurações.

# Instalação
 * No ```/etc/ansible``` tem o ```hosts``` que é um modelo de arquivo de inventário onde ficam os hosts que o server controla e o ```ansible.cfg``` é o arquivo de configuração do ansible.
 * Comandos adhoc: Você está na máquina do ansible e pede para ele executar um comando ou uma função nas máquinas que estão nos hosts. 
 
 Isso é diferente dos playbooks, que é uma porção de código e tarefas para serem executadas pelas máquinas.
 * Módulo apt: 
 ```ansible -b -i hosts elliot-01 -m apt -a "name=cmatrix state=present"```, ```-b``` é para da a permissão de sudo, vem de become user; ```-i``` hosts é pra indicar o arquivo de hosts e elliot-01 é a máquina a ser seleconada dentro do hosts; ```-m``` apt é a seleção do módulo apt; -a é o argumento do apt. Aqui não foi usado, mas o ```-k``` é para passar uma senha SSH no ansible.

 * Como executar um playbook: ```ansible-playbook -i hosts nome_playbook.yml```
 

 * Como eliminar que uma nova máquina peça a confirmação para adicionar a sua authorized_keys no host que está acessando a máquina

 ```export ANSIBLE_HOST_KEY_CHECKING=False ```

 * Erro apresentado: ```Permission denied (publickey).", "unreachable": true}```

 Solução: Ir na pasta que está o .pem em seguida ```ssh-agent zsh```
 ``` ssh-add name.pem```

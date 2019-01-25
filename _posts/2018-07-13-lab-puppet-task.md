---
layout: post
title: Laboratório de Puppet Task
subtitle: Estudo e testes do Puppet Task
gh-repo: clodonil/clodonil.github.io
gh-badge: [star, fork, follow]
tags: [vagrant, Puppet, Plan, Tasks, Git]
comments: true
---

Em outubro de 2017 a Puppet apresentou a sua nova aposta, o Puppet Tasks. Com isso a Puppet tenta cobrir uma lacuna que sua ferramenta apresentava com relação a workflow de tarefas.

Nesse Lab, vamos seguir o seguinte conteúdo para conhecer sobre o puppet task.

**Indice**
1. [Introdução ao Puppet Task](#p1)
  1.1. [Instalação no Centos 7](#p1.1)
  1.2. [Instalação no Windows](#p1.2)
  1.3. [Conhecendo o bolt](#p1.3)
2. [Criação e definição de nodes](#p2)
  2.1. [Provisionando em Vagrant](#p2.1)
  2.2. [Provisionando em Docker](#p2.2)
3. [Manipulando os nodes](#p3)
  3.1. [Rodando comandos Shell nos nodes Linux](#p3.1)
  3.2. [Rodando comandos PowerShell no node Windows](#p3.2)
  3.3. [Rodando Scripts no Linux](#p3.3)
  3.4. [Rodando Scripts no Windows](#p3.4)
  3.5. [Upload de Arquivo](#p3.6)
  3.6. [Usando Package e Services](#p3.6)
4. [Escrevendo as Tasks](#p4)
  4.1. [Tasks com parâmetros](#p1.2)
  4.2. [Controlando parâmetros com metadata](#p4.2)
  4.3. [Tasks com tratamento de error](#p4.3)
5. [Escrevendo os Plans](#p5)
6. [Referência](#p6)

Todos os códigos estão no repostório do [git](https://github.com/clodonil/puppet-tasks.git).


# 1. Introdução ao Puppet Task
Puppet é uma ferramente de gestão de configurações. Ela mantem o estado desejado de um servidor. 

O Puppet garante que o resultado/estado final é o desejado, mais não garante a ordem da execução das tarefas. 

Para controlar as ordens de execução é necessário utilizar recursos tais como `before/require` e `notify/subscribe` o que é praticamente enviável em ambientes complexos.

Visando essa lacuna a Puppet lançou o recursos de Tasks, também conhecido com o Puppet Tasks.

Basicamente o Puppet Tasks é um conjunto de funções, tais como executar comandos remotamente, enviar arquivos, executar scripts, executar tarefas, gerenciar serviços  e instalar pacotes.

Todas essas funções podem ser executadas ad-hoc, diretamente em um node.

Além de executar ad-hoc os comandos, é possível criar um Plan que orquestra um conjunto de funções de forma ordenada, criando uma infinidades de possibilidades, e mais diretamente um forma poderosa de fazer deploy.

A Puppet disponibilizou no Puppet Enterprise, apenas os recursos de tasks, services e package. E também lançou uma ferramenta opensource chamado puppet-bolt que possuem todos os outros recursos e utiliza o ssh/winrm para se comunicar com os nodes.

Portanto para esse estudo estaremos utilizando o bolt.

Para execução dos Plans e das Tasks não é necessário ter instalado o Puppet e nem o Puppet Enterprise, apenas o bolt.

O computador com o bolt instalado recebe o nome de manager, nenhum outro recurso do Puppet é necessário para execução desse lab. 

Todas as tasks e plans escritos para serem executados com o bolt, mantém 100% de compatibilidade com o Puppet Enterprise.

## 1.1 Instalação no Centos 7
Para instalação no Centos 7, primeiramente instale o repositório do  puppet e em seguida instalar o puppet-bolt.

```bash
$ rpm -ivh http://yum.puppet.com/puppet5/puppet5-release-el-7.noarch.rpm
$ yum install puppet-bolt
```

## 1.2 Instalação no Windows
No windows é possível instalar o pacote compilado msi através do seguinte link Puppet-Bolt.

Também é possível instalar através do chocolaty:

```bash
C:\> choco install Ruby
C:\> refreshenv
C:\> gem install puppe-bolt
```

## 1.3. Conhecendo o Bolt

Com o Bolt instalado execute o `bolt --help` para mostrar todas as opções do bolt.

Os principais recursos são:

| Comando | Descrição |
|----------------|----------|
|Bolt command run|	Executa comandos remotamente|
|bolt file upload	|Upload de arquivos|
|bolt script run	|Upload do script e executa remotamente|
|bolt task show	|Mostra a lista das tasks disponíveis|
|bolt task <task> |show	Mostra a documentação de uma task|
|bolt task run	|Executa uma task puppet|
|bolt plan show	|Mostra a lista de plan disponível|
|bolt plan <plan>| show	Mostra a documentação de um plan|
|bolt plan run	| Executa um Plan escrito em Puppet|

Opções:
| Descrição |
|----|----|
|-n, --nodes	|Identifica o node alvo|
|--noop	|Executa uma tasks/comando/script em modo noop (simulação)|

Autenticação

| Parametro |Descrição|
|-------|----|
|-u, --user|	Usuário para autenticação SSH/Winrm|
|-p, --password |	Senha do usuário para autenticação
|--private-key	|Chaves SSH para autenticação|

Elevação de acesso:

--run-as	Usuário que será utilizado para elevação de acesso.
--sudo-password	Senha para elevação de acesso com sudo
Parâmetros de execução:

-c, --concurrency	Número máximo de conexão simultânea (default 100)
--modulepath	Lista de diretórios dos módulos, separados por ",".
--configfile	Especifica o path do arquivo de configuração do bolt (default ~/.puppetlabs/bolt.yaml
--inventoryfile	Especifica o path do arquivo de inventário. (default ~/.puppetlabs/bolt/inventory.yaml
Transporte

--transport	Define o protocolo de transporte ssh, winrm,pcp, local;
--connect-timeout	Timeout de conexão
--[no--]tty	Ativa o pseudo terminal.
--tmpdir	Define o arquivo temporário para upload dos arquivos para execução nos nodes.
Muitos dos parâmetros podem ser configurados diretamente no arquivo do bolt, simplificando a execução do comando.

Um exemplo do arquivo ~/.puppetlabs/bolt.yaml:

modulepath: "/opt/modules:/etc/puppet/code/modules"
inventoryfile: "~/.puppetlabs/bolt/inventory.yaml"
format: human
ssh:
   host-key-check: false
   private-key: ~/.ssh/bolt_id

log:
   console:
       level: info
       ~/.bolt/debug.log:
            level: debug
            append: false
2.  Criação e definição de nodes
Para realização deste laboratório será utilizado 2 nodes em Linux (Centos 7) e um node Windows. Nos servidores Linux a comunicação será realizada utilizando o SSH e no Windows o WinRM.

Existe várias formas de realizar o provisionamento dos nodes, nesse lab usaremos o provisionamento em Docker e Vagrant com Virtualbox. O adotado para o restante do lab será o vagrant.

2.1 Provisionando em Vagrant
Considerando que o virtualbox e o vagrant está instalado, crie o Vagrantfile no diretório ~/puppet-task com o seguinte conteúdo.

# -*- mode: ruby -*-

Vagrant.configure("2") do |config|

  #  Configure base
  config.vm.box = 'centos/7'
  config.ssh.forward_agent = true
  config.ssh.insert_key = false

  # nodes
  $node_count = 2
  (1..$node_count).each do |id|
     config.vm.define "node#{id}" do |nodes|
         nodes.vm.hostname = "node#{id}"
         nodes.vm.network :private_network, :ip => "10.20.1.#{10+id}"
         nodes.vm.provision :hosts, :sync_hosts => true

    end
 end

  # Node of Windows
  config.vm.define :windows do |windows|
    windows.vm.box = "mwrock/WindowsNano"
    windows.vm.guest = :windows
    windows.vm.communicator = "winrm"
    windows.vm.network "private_network", ip: "10.20.1.20"
    windows.vm.provision :hosts, :sync_hosts => true
  end

 # Puppet Tasks - Bolt
 config.vm.define :bolt do |bolt|
    bolt.vm.hostname = "bolt"
    bolt.vm.network :private_network, :ip => "10.20.1.100"
    bolt.vm.provision :hosts, :sync_hosts => true

    # Install of Puppet Bolt
    bolt.vm.provision "shell", inline: "rpm -ivh http://yum.puppet.com/puppet5/puppet5-release-el-7.noarch.rpm &&  yum -y install puppet-bolt"
  end
end
Os seguintes plugins do vagrant devem estar instalados:

vagrant-hosts (2.8.2)
winrm (2.2.3)
winrm-elevated (1.1.0)
winrm-fs (1.2.0)
Uma rápida explicação sobre o Vagrantfile, caso você não esteja familiarizado com essa tecnologia. A variável "$node_count" define a quantidade de nodes que serão criadas do centos 7 e a comunicação via SSH. Também está sendo provisionado uma máquina Windows (Nano Server) com a comunicação WinRM.

A vm "bolt" será utilizado como manager para gerenciar os outros nodes. Para isso é necessário que essa vm acesse todas as outras. Para tornar isso possível, copie a chave de acesso para a pasta do puppet-task:

$ cp ~/.vagrant.d/insecure_private_key /puppet-task/
Para aplicar a configuração do vagrantfile e criar as máquinas virtuais, execute o comando:

$ vagrant up node1 node2 windows bolt
Uma forma rápida de acesso com o SSH ao nodes provisionados é criando um arquivo de ssh-config.

Para isso acesse a vm bolt ("vagrant ssh bolt"), e como root crie o arquivo "generate-ssh.sh" e execute para criar o arquivo /root/.ssh/config.

#!/bin/bash

echo "" > /root/.ssh/config

while read line
do
   ip=$(echo $line | awk {'print $1'})
   host=$(echo $line | awk {'print $2'})

   echo "Host $host" >> /root/.ssh/config
   echo "    HostName $ip" >> /root/.ssh/config
   echo "    User vagrant" >> /root/.ssh/config
   echo "    Port 22" >> /root/.ssh/config
   echo "    PasswordAuthentication no" >> /root/.ssh/config
   echo "    IdentityFile /vagrant/insecure_private_key" >> /root/.ssh/config
   echo "    IdentitiesOnly yes" >> /root/.ssh/config
   echo "    LogLevel FATAL" >> /root/.ssh/config
   echo "    ForwardAgent yes"   >> /root/.ssh/config
done  < /etc/hosts
Executando o arquivo:

$ chmod +x generate-ssh.sh
$ ./generate-ssh.sh
Para testar a conexão com os nodes:

$ ssh node1
$ ssh node2
2.2 Provisionando em Docker
Usando o docker também é possível provisionar os servidores Linux de forma rápida. Para isso edit o arquivo "docker-compose.yml " e tenha instalado o docker.

version: '3'
services:
  ssh:
    image: rastasheep/ubuntu-sshd
    ports:
      - 22
Execute o seguinte comando para provisionar o servidor ubuntu com SSH.

$ docker-compose up -d
Caso você queira mais servidores, pode utilizar o --scale para determinar a quantidade de nodes.

$ docker-compose up --scale ssh=3 --detach
Não tem importância a forma de provisionar, basicamente os nodes Linux precisam ter SSH e WinRM nos servidores Windows. Portanto provisione da forma que você achar melhor.

3.  Manipulando os nodes
Nos nodes provisionados é possível executar comandos, fazer upload de arquivos, rodar scripts e executar tasks e Plans.

Tasks são tarefas (scripts) que serão executada nos nodes. A principal diferença entre as tasks e scripts é que as tasks recebem parâmetros e os parâmetros podem podem ser controlados.

3.1  Rodando comandos shell nos nodes Linux
Para executar comandos ad-hoc do bolt:

Sintaxe:

$ bolt command run <comando> --nodes <nodes>

Como exemplo, é executado primeiramente o comando 'uptime' para retornar o tempo que o servidor está trabalhando:

$ bolt command run 'uptime' --nodes node1
Started on node1...
Finished on node1:
  STDOUT:
     13:30:04 up 14:56,  0 users,  load average: 0.02, 0.03, 0.05
Successful on 1 node: node1
Ran on 1 node in 0.72 seconds
Em caso de erro 'Host key verification failled', existe um conflito de chaves do SSH, é preciso apagar as entradas relacionadas no arquivo ~/.ssh/sshd_hosts.

Também é possível executar comandos em múltiplos nodes ao mesmo tempo.

$ bolt command run 'uptime' --nodes node1,node2
Started on node1...
Started on node2...
Finished on node1:
  STDOUT:
     13:30:37 up 14:57,  0 users,  load average: 0.01, 0.03, 0.05
Finished on node2:
  STDOUT:
     13:30:37 up 14:55,  0 users,  load average: 0.00, 0.01, 0.05
Successful on 2 nodes: node1,node2
Ran on 2 nodes in 0.71 seconds
Para grandes números de nodes ou para agrupa-lós é necessário a criação do arquivo de inventory.yaml. Crie esse arquivo no diretório ~/.puppetlabs/bolt.

nodes: [ node1, node2 ]
Em casos específicos, os nodes podem solicitar senha para autenticação, nesse caso é possível passar as informações ad-hoc na execução do bolt ou através da inclusão dos acessos no arquivo inventory.yaml.

Sintaxe:

$bolt  command run <comando> --nodes all --user <login> --password <senha>

E no inventory:

Nodes: [node1, node2, node3] 
    config: 
         transports: 
              ssh: 
                 user    : <login> 
                 password: <senha>


3.2 Rodando comandos PowerShell no node Windows
O bolt executa os comandos no Windows através do WinRM, conforme já mencionado anteriormente.

Sintaxe:

$ bolt command run < comando> --nodes winrm://<node> --user <login> --password <senha>

No parâmetros WinRM é informado o endereço do servidor Windows. E caso seja necessário informar o usuário e senha, utilize o "--user" e o "--password".

O bolt por padrão ativa a comunicação com SSL para comunicação com o WinRM, caso o servidor Windows não esteja com o SSL ativo, a conexão não vai ser realizado e uma mensagem de erro de comunicação será apresentado ("Unknown Protocol").

Para resolver esse erro é necessário ativar o SSL no WinRM no servidor Windows ou desativar a utilização do SSL pelo bolt através da opção --no-ssl.

Para simplificar a sintaxe do comando, será criado uma variável "WIN" com o endereçamento do WinRM.

WIN="winrm://vagrant:vagrant@windows:5985"
A chamada do comando bolt fica mais limpa:

$ bolt command run "gps | select ProcessName" --no-ssl --nodes $WIN
Started on windows...
Finished on windows:
  STDOUT:
    ProcessName
    -----------
    csrss
    EMT
    Idle
    lsass
    services
    smss
    svchost
    svchost
    svchost
    svchost
    svchost
3.3 Rodando scripts no Linux
Com o bolt é possível executar qualquer tipo de scripts, desde o compilador/interpretador esteja presente no node. Perceba que não é necessário ter o interpretador no servidor manager.

A sintaxe para executar scripts:

$ bolt script run <script> --nodes <nodes>

Nesse Lab vamos utilizar, bash para Linux e PowerShell para Windows.

Como exemplo de execução de script, será executado um harderning do Centos7.

Faça o download do script no diretório /root/scripts. É necessário criar esse diretório.

$ curl -O https://raw.githubusercontent.com/naingyeminn/CentOS7_Lockdown/master/centos7.sh
O script de hardening aplica um conjunto de configurações para tornar o linux mais seguro. Devido o script alterar arquivos protegidos, é necessário ter acesso de root nas VMs. Dessa forma, informa o parâmetro "--run-as" para elevação de acesso:

$ bolt script run /root/script/centos7.sh --run-as root -n all
Started on node2...
Started on node1...
Finished on node2:
  STDOUT:
    Disabling Legacy Filesystems
    Removing GCC compiler...
    Loaded plugins: fastestmirror
    No Packages marked for removal
    Removing legacy services...
    Disabling LDAP...
    Disabling DNS...
    Disabling FTP Server...
    Disabling Dovecot...
    Disabling Samba...
    Disabling HTTP Proxy Server...
Ao executar o script pelo bolt, primeiramente é feito o download nos nodes antes de executar.

3.4 Rodando scripts no Windows
Todos os administradores de sistema já têm um conjunto de scripts que executa nos servidores. Esses scripts podem ser reutilizados e serem facilmente aplicados em um conjunto de servidores. Caso você tenha um desses scripts pode utiliza-ló. Nesse exemplo vamos utilizar um script simples para teste de conexão.

Salve o conteúdo a seguir com o nome '/root/script/testconnection.ps1":

Test-Connection -ComputerName "example.com" -Count 3 -Delay 2 -TTL 255 -BufferSize 256 -ThrottleLimit 32
Executando o script:

$ bolt script run /root/script/testconnection.ps1 --no-ssl -n $WIN
3.5 Upload de arquivo
Um dos recursos normalmente utilizado na administração de servidores é o upload de arquivos. Normalmente durante um deploy é necessário enviar o pacote da aplicação para os servidores ou um conjunto de configuração conforme o ambiente.

A sintaxe do upload de arquivo:

$ bolt file /local_dir/file /server_dir/file  --nodes  all

Para enviar uma atualização de pagina html para o servidor nginx, fica dessa forma:

Crie o arquivo /root/files/index.html com qualquer conteúdo.

$ bolt file upload /root/files/index.html /tmp/index.html --nodes node1
Started on node1...
Finished on node1:
  Uploaded '/root/files/index.html' to 'node1:/tmp/index.html'
Successful on 1 node: node1
Ran on 1 node in 1.14 seconds
3.6 Usando Package
Através do bolt é possível gerenciar pacotes e servidores. No exemplo a seguir é instalado o pacote "httpd" no node node1. As opções possíveis do parâmetro "action" são "install/status/unistall/upgrade".

$ bolt task run package action => 'install', name => 'httpd', version => latest --nodes web
Além de gerenciar os pacotes, também é possível gerenciar os serviços. As opções do parâmetros action são "start/stop/restart/enable/disable/status".

$ bolt task run service action => status  name => httpd --nodes all
Para o bolt utilizaros pacotes e módulos do Forge, é necessário instalar a ferramenta r10k.

4. Escrevendo Tasks
Igualmente aos scripts, as tasks podem ser escritos em qualquer linguagem de programação, desde que seja possível executar no node.

As tasks devem ser utilizadas como um módulo do Puppet. Isso significa que podem ser manipuladas pelo Forge e também gerenciadas pelas ferramentas existentes do Puppet.

Por padrão as tasks recebem argumentos como variável de ambiente prefixadas com PT (Puppet Tasks), tais como "$PT_exemplo".

As tasks devem ser criadas no diretório "tasks" dentro do módulo. Um módulo pode ter várias tasks, mas a tasks "init" é especial. Será executada caso nenhum nome de tasks seja passado durante a chamada do módulo.

Por exemplo, o módulo "exercicio1" e a tasks deploy em  "exercicio1/tasks/deploy.sh", tem o nome composto de exercicio1::deploy. O nome funciona como referência a tasks para ser executada.

As tasks também pode ter metadata que valida os argumentos de entrada e controla como as tasks vai ser executada. A definição das entradas de parâmetros no metadata também é uma forma de reduzir o risco de segurança.

Como exemplo de tasks, crie o arquivo init.sh no diretório /modules/exercicio1/tasks:

#!/bin/bash
interface=$PT_interface
ip=$(/sbin/ip add show $interface | grep "inet\b" | awk '{print $2}' | cut -d / -f 1 )
echo $ip
Para executar a tasks padrão (init). Utilize o seguinte comando:

$ bolt task run exercicio1 interface=eth0 --nodes all --modulepath /modules/
Started on node1...
Started on node2...
Finished on node1:
  eth0:10.0.2.15
  {
  }
Finished on node2:
  eth0:10.0.2.15
  {
  }
Successful on 2 nodes: node1,node2
Ran on 2 nodes in 1.21 seconds
Como o nome da tasks é "init", foi passado apenas o nome do módulo com o argumento interface  que dentro do script  fica como "PT_interface".

Tasks mais elaboradas desenvolvidas em outras linguagens como Python e Ruby podem retornar dados mais estruturados (não estou dizendo que em bash não seja possível).

Usando o exemplo do módulo exercicio1, crie mais uma tasks com o nome gethost.py com o seguinte conteúdo.

#!/usr/bin/env python

import socket
import sys
import os
import json

host = os.environ.get('PT_host')
result = { 'host': host }

if host:
    result['ipaddr'] = socket.gethostbyname(host)
    result['hostname'] = socket.gethostname()
    # The _output key is special and used by bolt to display a human readable summary
    result['_output'] = "%s is available at %s on %s" % (host, result['ipaddr'], result['hostname'])
    print(json.dumps(result))
else:
    # The _error key is special. Bolt will print the 'msg' in the error for the user.
    result['_error'] = { 'msg': 'No host argument passed', 'kind': 'exercise5/missing_parameter' }
    print(json.dumps(result))
    sys.exit(1)
Pode executar a tasks da seguinte forma:

bolt task run exercicio1::gethost host="google.com" --nodes all --modulepath /modules/
Started on node2...
Started on node1...
Finished on node2:
  google.com is available at 216.58.202.238 on node2
  {
    "host": "google.com",
    "hostname": "node2",
    "ipaddr": "216.58.202.238"
  }
Finished on node1:
  google.com is available at 172.217.30.110 on node1
  {
    "host": "google.com",
    "hostname": "node1",
    "ipaddr": "172.217.30.110"
  }
Successful on 2 nodes: node1,node2
Ran on 2 nodes in 1.20 seconds
Nesse exemplo um pré-requisito para execução do script é a presença do python no nodes.

4.1 Tasks com Parâmetros
As tasks recebem os parâmetros basicamente através de variável de ambiente (environment) ou através de entrada padrão (stdin). A escolha do método de passagem de parâmetros  interfere diretamente na escrita do metadata e também no desenvolvido do próprio script.

A seguir temos alguns exemplos de recebimento de parâmetros em bash, ruby, python e PowerShell com passagem de parâmetros em stdin e environment. Para esses exemplos, crie o diretório exercicio2 no diretório modules.

Bash
Tasks em bash com env:
#!/bin/bash
#filename: tasks_env_bash.sh

params=$PT_params
echo $params
Tasks em bash com stdin:

#!/bin/bash
#filename: tasks_stdin_bash.sh

read params
echo $params
Comando para execução da tasks:

$ bolt task run exercicio2::tasks_env_bash params=10 --nodes all --modulepath /modules/

 Python
Tasks em python com env:
#!/usr/bin/env python
#filename: tasks_env_python.py

import os
host = os.environ.get('PT_host')
print(host)
Tasks em python com stdin:

#!/usr/bin/python
#filename: tasks_stdin_python.py

host=input()
print(host)
Comando para execução da tasks:

$ bolt task run exercicio2::tasks_env_python host="server1" --nodes all --modulepath /modules

 Ruby
Tasks em ruby com env:
#!/usr/bin/ruby
#filename: tasks_env_ruby.rb

params = ENV["PT_valor"]
puts("Parâmetro: #{params}")
Tasks em ruby com stdin:

#!/usr/bin/ruby
#filename: tasks_stdin_ruby.rb

params = STDIN.read
puts("Parâmetros: #{params}")
Comando para executar a tasks Ruby:
$ bolt task run exercicio2::tasks_env_ruby valor=10 --nodes all --modulepath /modules

PowerShell
Tasks em PowerShell
[CmdletBinding()]
Param(
  [Parameter(Mandatory = $False)]
 [String]
  $Name
  )

if ($Name -eq $null -or $Name -eq "") {
  Get-Process
} else {
  $processes = Get-Process -Name $Name
  $result = @()
  foreach ($process in $processes) {
    $result += @{"Name" = $process.ProcessName;
                 "CPU" = $process.CPU;
                 "Memory" = $process.WorkingSet;
                 "Path" = $process.Path;
                 "Id" = $process.Id}
  }
  if ($result.Count -eq 1) {
    ConvertTo-Json -InputObject $result[0] -Compress
  } elseif ($result.Count -gt 1) {
    ConvertTo-Json -InputObject @{"_items" = $result} -Compress
  }
}
Sintaxe para execução da tasks no windows:
$ bolt task run exercicio2::tasks_powershell --no-ssl --nodes $WIN --modulepath /modules

$ bolt task run exercicio2::tasks_powershell name=System --no-ssl --nodes $WIN --modulepath /modules

Sempre estamos passado o caminho que estão localizados os módulos, entretanto podemos adicionar essa informação no arquivo ~/.puppetlabs/bolt/bolt.yaml.

modulepath: "/modules/"
4.2 Controlando parâmetros com metadata
Com os metadata é feito a documentação da tasks e também o controle dos parâmetros de entrada. O controle de parâmetros é importante para diminuir o risco de entrada de valores não desejáveis.

A tabela a seguir, mostra os parâmetros e os valores padrões da configuração do arquivo de metadata.

Chave	Descrição	Valor	Default
description	Descrição da função da tasks	String	None
puppet_task_version	A versão da tasks	Integer	1
supports_noop	Quando a tasks suporta noop	Boolean	False
input_method	As opções de passagem de parâmetros para a tasks	environment
stdin
powershell	stdin
parameters	Tipo de dados do Puppet	Tipo Puppet	None
O metadata podem aceitar muitos tipos de dados no campo "parameters". A tabela seguinte mostra os mais comuns usadnos por tasks.

Tipo	Descrição
String	Aceita qualquer String.
String[1]	Aceita qualquer String não vazia.
Enun[opcao1, opcao2]	Aceita parâmetros de uma lista possível
Pattern[/\A\W+\Z/]	Aceita String que validas pelas expressão regular [/\A\W+\Z/]
Integer	Aceita valores Inteiros.
Optional[String[1]]	Aceita a opção de uma lista e também permite valores nulos.
Array[String]	Aceita um Array de String.
Hash	Aceita um objeto Json.
Variant[Integer, Pattern[/\A\W+\Z/]]	Aceita valores que aceita a expressão regular.
O script "install.sh", exemplifica a utilização do metadata. O script basicamente faz a instalação do openjdk com a opção de desinstalar a versão antiga.Pelo metadata deve ser realizado os seguintes controles:

A versão do openjdk deve ser homologada anteriormente;
A opção de desinstalação seja binária (yes ou  no);
O script é feito normalmente, sem nenhum controle de parâmetro; Crie o diretório /modules/exercicio3/tasks.

#!/usr/bin/bash
#install.sh
install="no"
uninstall="no"
version=$(echo $PT_version | sed s/_/./g)
uninstall=$PT_uninstall
old=$(rpm -qa | grep "openjdk")
pacote="java-$version-openjdk"
yum install -y $pacote &> /dev/null

if [ "$?" == "0" ]; then
   echo "Instalado $pacote"
else
  echo "Problema na Instalado do $pacote"
fi

if [ "$uninstall" == "yes" ]; then
  rpm -e $old
  if [ "$?" == "0" ]; then
     echo "Desinstalado $old"
  else
     echo "Problema na Desinstalacao do $pacote"
  fi
fi
O arquivo de JSON deve ter o mesmo nome do script, "install.json". A opção "description" descreve o funcionamento do script e no campo "parameters" é possível controlar as entradas.

No exemplo, o parâmetro "version" obrigatoriamente deve ter um dos valores da lista (1_6_0, 1_7_0, 1_8), Igualmente o parâmetros "uninstall" deve ser "yes' ou "no".

Coloquei as versão do java (1_6_0, 1_7_0...), porque no arquivo de metadata na versão da elaboração desse LAB não aceitava o ponto "(1.6.0, 1.7.0...)" e a troca do "_" pelo "." foi realizado no script.

Segue o arquivo do metadata install_openjdk.json.

{
  "description" : "Instalação e Desinstalação do OpenJDK",
  "input_method": "environment",
  "parameters": {
        "version": {
              "description": "Versão do Java Homologado para instalação",
              "type": "Enum['1_6_0','1_7_0','1_8_0']"
         },
        "uninstall":{
               "description": "Desinstall versões antigas",
               "type": "Enum[yes,no]"
         }
   }
}
Rodando o script:

bolt task run exercicio3::install version=1_8_0 uninstall=no --nodes all
Started on node2...
Started on node1...
Finished on node2:
  Instalado java-1.8.0-openjdk
  {
  }
Finished on node1:
  Instalado java-1.8.0-openjdk
  {
  }
Successful on 2 nodes: node1,node2
Ran on 2 nodes in 2.01 seconds
4.3 Tasks com tratamento de erro
Quando uma task é finalizada com um código diferente de zero, o puppet-tasks verifica uma chave de erro ("_error"). Se essa chave não estiver presente, um erro genérico será adicionado ao campo de retorno. Se não houver texto na saída padrão (stdout), mas  se tiver presente no stderr, o texto stderr será incluído na mensagem.

A estrutura da chave de erro:

{
  "_error": {
         "msg"   : "Falha na task",
         "king"  : "puppetlabs.tasks/tasks-errror",
         "details: " { "exitcode" :1 }
  }
}
A estrutura da mensagem de erro é composta dos seguintes campos:

msg: Mensagem amigável sobre o erro,
kind: Mensagem mais detalhada e avançada. Mensagem pode ser compartilhada entre os módulos.
details: Um objeto estruturado sobre a tasks.
A seguir um exemplo da utilização de tratamento de erro:

#!/usr/bin/env python

import os
import json

file = os.environ.get('PT_filename')

try:
   open_file = open(filename, "r")
   conteudo  = open_file.readlines()
   open_file.close()
   print(conteudo)
except:
    result = {
               "msg" : "Problema ao abrir o arquivo {0}".format(filename),
               "kind": "myproj/task-error",
               "details": {"exitcode": 1},
             }
    print(json.dumps(result))
5. Escrevendo Plan
O Plan é escrito utilizando a linguagem do Puppet, e nele pode ser vinculado a um conjunto de comandos, scripts, pacotes, tasks e a outros plan, facilitando  a reutilização  de código. Os plan executa a lógica localmente e os comandos/scripts/tasks são executados remotamente.

Apesar do plan ser escrito utilizando a linguagem do Puppet, não é necessário ter o puppet instalado no manager.

O plan pode executar as seguintes funções:

run_command: Roda um comando em um ou mais nodes;
file_upload: Faz upload de arquivos para um ou mais nodes;
run_script: Roda um script ( não é uma tasks);
run_tasks: Roda uma task em um ou mais nodes;
run_plan: Roda um plan em um ou mais nodes;
Vamos para um exemplo simples para entender como funciona o plan. Para isso crie o seguinte arquivo /modules/exercicio4/plans/comandos.pp.

plan exercicio4::comandos (TargetSpec $nodes){
    run_command("uptime", $nodes)
}
Nesse exemplo o plan basicamente executa um comando.

Para rodar o plan:

$ bolt plan run exercicio4::comandos --nodes all
Starting: command 'uptime' on node1, node2
Finished: command 'uptime' with 0 failures in 0.78 sec
Plan completed successfully with no result
Perceba que os nodes são passados pelo TargetSpec, além dos nodes outros parâmetros podem ser utilizado.

Além de comandos é possível executar as tasks e incluir lógica/controle de verificação dentro dos plans, conforme mostra o exemplo a seguir:

plan my_app::deploy (
    String[1] $version,
    Enum[dev, staging,prod] $tier = 'dev',
){
   $app_nodes = my_app::get_nodes($tier, 'app')
   $db_nodes  = my_app::get_nodes($tier, 'db')

   $update_db = run_tasks('my_db::update',$db_nodes, version => $version)
   $update_app = run_tasks('my_app::update',$app_nodes, version => $version)

   if ($update_db.ok && $update_app.ok){
      $deploy_result = run_task('my_app::deploy', $app_nodes, version => $version)
      if ($deploy_result.ok) {
          notice("Deploy finalizado com sucesso nos servidores ${nodes} para a versão ${version}")
      } else {
          notice("Falha no deploy para a versão ${version} nos servidores ${nodes.erros.keys}")
      }
   } else {
     notice("Deploy finalizado com falha")
   }
}
Uma tasks executada dentro do Plan, retorna os seguintes objetos:

Retorno	Descrição
names	Retorna uma lista de nodes envolvidos;
empty	Retorna true, se o retorno estiver vazio;
count	Retorna um inteiro com o número de nodes envolvidos na tasks;
first	Retorna um objeto com todas as informações da tasks:
find	Pesquisa o retorno de um node especifico;
error_set	Retorna o erro, caso exista;
targets	Retorna uma lista com todos os retornos dos nodes
ok	Retorna true se não tiver nenhum erro.

# 6. Referência
- [https://puppet.com/docs/bolt](https://puppet.com/docs/bolt)
- [https://github.com/puppetlabs/bolt](https://github.com/puppetlabs/bolt)
- [https://github.com/puppetlabs/tasks-hands-on-lab](https://github.com/puppetlabs/tasks-hands-on-lab)
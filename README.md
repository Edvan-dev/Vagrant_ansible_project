
# Configuração de Ambiente com Vagrant e Ansible

### Projeto desenvolvido em cumprimento a avalição da disciplina Administração de Sistemas Abertos.

Aluno: Edvan da Silva Oliveira

Prof: Pedro Batista de Carvalho Filho

Este projeto utiliza o Vagrant para configurar e provisionar uma máquina virtual baseando-se na distribuição Debian 12, com configuração de rede privada, discos adicionais e provisionamento automatizado com Ansible.

## Requisitos

Certifique-se de que os seguintes softwares estejam instalados no seu sistema:

- [Vagrant](https://www.vagrantup.com/)
- [VirtualBox](https://www.virtualbox.org/)
- [Ansible](https://www.ansible.com/)
- Módulo passlib instalado no python
- comando: pip3 install --user passlib


## Configuração do Ambiente

O arquivo `Vagrantfile` define as configurações da máquina virtual:

1. **Definição da box base**:
   ```ruby
   config.vm.box = "debian/bookworm64"
   ```
   Define a box base utilizada como Debian 12 genérico.

2. **Configuração da máquina virtual**:
   ```ruby
   config.vm.provider "virtualbox" do |vb|
     vb.name = "p01_Edvan"
     vb.memory = 1024
   end
   ```
   - Nome da máquina virtual: `p01_Edvan`
   - Memória RAM alocada: 1024 MB

3. **Configuração de rede**:
   ```ruby
   config.vm.network "private_network", ip: "192.168.57.10"
   ```
   Define uma rede privada com o endereço IP fixo `192.168.57.10`.

4. **Configuração de discos adicionais**:
   ```ruby
   (1..3).each do |i|
     config.vm.provider "virtualbox" do |vb|
       vb.customize ["createhd", "--filename", "disk#{i}.vdi", "--size", 10240]
       vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", i, "--device", 0, "--type", "hdd", "--medium", "disk#{i}.vdi"]
     end
   end
   ```
   - Cria três discos virtuais (disk1.vdi, disk2.vdi, disk3.vdi) com 10 GB cada.
   - Anexa os discos ao controlador SATA da máquina virtual.

5. **Provisionamento com Ansible**:
   ```ruby
   config.vm.provision "ansible" do |ansible|
     ansible.playbook = "provisionar.yml"
   end
   ```
   Utiliza o Ansible para provisionar a máquina virtual utilizando o playbook `provisionar.yml`.

## Funcionalidades do Playbook

O playbook `provisionar.yml` realiza as seguintes tarefas:

### Configurações Iniciais
- Atualiza todos os pacotes do sistema.
- Define o hostname da máquina virtual.

### Gerenciamento de Usuários e Grupos
- Cria os usuários `edvan`, `oliveira`, `ifpb` e `nfs-ifpb` com senhas e shells personalizados.
  - **Usuários e suas senhas:**
    - `edvan`: Senha `ifpb`
    - `oliveira`: Senha `ifpb`
    - `ifpb`: Senha `admin`
    - `nfs-ifpb`: Sem senha configurada (shell desabilitado)
- Configura diretórios `.ssh` e gera pares de chaves SSH para os usuários.
- Cria os grupos `ifpb` e `acesso_ssh` e adiciona os usuários apropriados a esses grupos.
- Adiciona o usuário `ifpb` ao grupo `sudo` para permissões administrativas.

### Configuração de Chaves SSH
- Verifica e gera chaves públicas locais.
- Adiciona as chaves públicas aos arquivos `authorized_keys` dos usuários para autenticação SSH.

### Configurações de Segurança SSH
- Atualiza o arquivo `sshd_config` para desativar o login do root, desativar a autenticação por senha e permitir apenas o grupo `acesso_ssh`.
- Configura uma mensagem de aviso de acesso no arquivo `/etc/motd`.
- Implementa um script de log de acessos em `/etc/profile.d/log_access.sh`.

### Gerenciamento de Discos e Volumes Lógicos
- Inicializa discos adicionais como volumes físicos e os agrupa em um grupo de volumes chamado `dados`.
- Cria um volume lógico chamado `sistema`, formata como `ext4` e monta no diretório `/dados`.

### Configuração do Servidor NFS
- Instala o servidor NFS e garante que o diretório `/dados/nfs` exista com permissões apropriadas.
- Configura o compartilhamento NFS para a rede `192.168.57.0/24`.
- Mapeia usuários anônimos para o usuário `nfs-ifpb`.

### Outras Configurações
- Configura o script de log de acessos para registrar eventos de login.
- Garante que o diretório `/dados/nfs/acessos` esteja acessível para os grupos relevantes.
- Aplica e reinicia serviços necessários (como o SSH e o NFS).

## Como Usar

1. Clone este repositório:
   ```bash
   git clone <URL_DO_REPOSITORIO>
   cd <PASTA_DO_REPOSITORIO>
   ```

2. Inicie a máquina virtual:
   ```bash
   vagrant up
   ```

3. Acesse a máquina virtual:
   ```bash
   ssh ifpb@192.168.57.10 #usuário com privilégio root
   ssh edvan@192.168.57.10 #usuário com níveis basicos 
   ```

4. Para aplicar ou atualizar as configurações de provisionamento:
   ```bash
   vagrant provision
   ```

5. Para desligar a máquina virtual:
   ```bash
   vagrant halt
   ```

6. Para destruir a máquina virtual e remover os discos:
   ```bash
   vagrant destroy
   ```

## Estrutura do Repositório

- `Vagrantfile`: Arquivo de configuração principal do Vagrant.
- `provisionar.yml`: Playbook do Ansible para provisionamento automatizado.

## Observações

- Certifique-se de que o Ansible esteja configurado corretamente no seu sistema.
- Os discos adicionais criados só serão anexados durante a inicialização da máquina virtual.

Para mais informações, consulte a [documentação oficial do Vagrant](https://www.vagrantup.com/docs).

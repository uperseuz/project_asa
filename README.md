Projeto DevOps com Vagrant e Ansible
Alunos: Caua e Joao

DISCIPLINA: Administração de Sistemas Abertos

PROFESSOR: Leonidas Lima

PERÍODO: 2025.2

Introdução
Este projeto tem o objetivo de automatizar o provisionamento e a configuração de uma infraestrutura com 4 máquinas virtuais Linux (Debian), utilizando Vagrant e Ansible. O ambiente simula um cenário de DevOps, integrando serviços os SSH, NFS, LVM, DHCP, Apache, DNS e MariaDB.

Vagrantfile
O projeto inclui um Vagrantfile que define a criação de quatro máquinas virtuais:

Servidor de Arquivos (arq):

Hostname: arq.caua.joao.devops

IP estático: 192.168.56.121 (XX = 21 da matrícula do Caua)

512 MB de RAM

Três discos adicionais de 10 GB cada

Servidor de Banco de Dados (db):

Hostname: db.caua.joao.devops

IP via DHCP com MAC: 0800271A0021

512 MB de RAM

Servidor de Aplicação (app):

Hostname: app.caua.joao.devops

IP via DHCP com MAC: 0800272B2100

512 MB de RAM

Cliente (cli):

Hostname: cli.caua.joao.devops

IP via DHCP

1024 MB de RAM

Todas as máquinas usam a box debian/bookworm64, com linked_clone habilitado, guest additions desabilitado e sem geração de novas chaves SSH. Um trigger é configurado para desabilitar o servidor DHCP do VirtualBox na rede host-only.

Playbooks

common.yml
Aplica configuração comum a todas as VMs:

Atualização do sistema (update e upgrade)

Instalação e configuração do Chrony (NTP) com servidor pool.ntp.br

Definição do timezone para America/Recife

Criação do grupo ifpb

Criação dos usuários caua e joao no grupo ifpb

Configuração de segurança do SSH:

Apenas autenticação por chaves públicas

Bloqueio de acesso root via SSH

Apenas usuários dos grupos vagrant e ifpb podem acessar

Banner de aviso de segurança

Geração de chaves SSH para os usuários

Instalação do cliente NFS

Permissão de sudo sem senha para o grupo ifpb

arq.yml
Configura o servidor de arquivos (arq) com todos os serviços:

DHCP (isc-dhcp-server):

Servidor autoritativo para rede 192.168.56.0/24

Faixa de IPs: 192.168.56.50 a 192.168.56.100

Lease time padrão: 180 segundos

Lease time máximo: 3600 segundos

Gateway padrão: 192.168.56.1

Domínio: caua.joao.devops

DNS (Bind9):

Aceita requisições da rede 192.168.56.0/24

Forwarders: 1.1.1.1 e 8.8.8.8

Zonas direta e reversa para caua.joao.devops

Registros A para arq (192.168.56.121), db (192.168.56.105), app (192.168.56.115)

LVM:

Utiliza 3 discos de 10GB para criar Volume Group dados

Cria Logical Volume ifpb com 15GB

Formata com ext4 e monta automaticamente em /dados

NFS Server:

Compartilha /dados/nfs para rede 192.168.56.0/24

Cria usuário nfs-ifpb sem shell (/usr/sbin/nologin)

Mapeia todos os usuários remotos para nfs-ifpb

Configura sync para gravações imediatas no disco

db.yml
Configura a VM db:

Instalação do servidor MariaDB (mariadb-server)

Inicialização e habilitação do serviço MariaDB

Instalação do autofs para montagem automática NFS

Configuração para montar /dados/nfs do servidor arq em /var/nfs

Inicialização e habilitação do serviço autofs

app.yml
Configura a VM app:

Instalação do servidor web Apache2

Criação de página web personalizada (index.html) com:

Descrição do projeto

Nomes e matrículas dos integrantes

Informações da infraestrutura

Instalação do autofs para montagem automática NFS

Configuração para montar /dados/nfs do servidor arq em /var/nfs

Inicialização e habilitação dos serviços Apache e autofs

cli.yml
Configura a VM cli:

Instalação dos pacotes firefox-esr e xauth

Configuração do SSH para permitir encaminhamento X11

Instalação do autofs para montagem automática NFS

Configuração para montar /dados/nfs do servidor arq em /var/nfs

Arquivos de Configuração
Para o servidor arq, são utilizados templates Jinja2:

dhcpd.conf.j2 - Configuração do servidor DHCP

named.conf.options.j2 - Opções do servidor DNS Bind9

named.conf.local.j2 - Configuração de zonas DNS

db.dominio.j2 - Zona direta para caua.joao.devops

db.reversa.j2 - Zona reversa para rede 192.168.56.0/24

Para o servidor app:

index.html.j2 - Template da página web do projeto



Uso: 

git clone https://github.com/uperseuz/project_asa.git
cd project_asa

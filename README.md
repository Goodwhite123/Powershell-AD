🛡️ Active Directory & Windows Server - Lab Automation Scripts

Este repositório contém uma coleção de scripts em PowerShell desenvolvidos para automatizar a criação e gestão de um ambiente de Active Directory (AD DS), aplicando as melhores práticas de mercado para organização e governança de acessos.

📋 Resumo dos Scripts

Criação de Estrutura de OUs

Implementação de Governança AGDLP

Configuração de Rede para Máquinas Clientes

1. Criação da Estrutura de OUs

Para evitar usuários e computadores dispersos nos contêineres padrão, este script cria uma estrutura lógica baseada em Unidades Organizacionais (OUs). Ele detecta automaticamente o domínio atual e cria uma OU raiz (Matriz) contendo sub-OUs para gerenciamento granular.

Onde rodar: Controlador de Domínio (Windows Server)

# Importa o módulo do Active Directory
Import-Module ActiveDirectory

# Obtém o Distinguished Name (DN) do domínio atual (ex: DC=cyberlab,DC=local)
$domainDN = (Get-ADDomain).DistinguishedName

# 1. Cria a OU raiz "Matriz" na raiz do domínio
New-ADOrganizationalUnit -Name "Matriz" -Path $domainDN

# Define o caminho (Path) da nova OU "Matriz" para criar as sub-OUs dentro dela
$matrizDN = "OU=Matriz,$domainDN"

# 2. Cria as sub-OUs organizacionais dentro da OU "Matriz"
New-ADOrganizationalUnit -Name "Usuarios" -Path $matrizDN
New-ADOrganizationalUnit -Name "Grupos" -Path $matrizDN
New-ADOrganizationalUnit -Name "Contas de Serviço" -Path $matrizDN
New-ADOrganizationalUnit -Name "Estações de Trabalho" -Path $matrizDN

Write-Host "Estrutura de OUs criada com sucesso!" -ForegroundColor Green



2. Governança de Acessos (AGDLP)

Script que implementa a arquitetura recomendada pela Microsoft: AGDLP (Accounts, Global Groups, Domain Local Groups, Permissions).
O script automatiza a criação de contas de usuários, grupos baseados em cargos (Global), grupos baseados em recursos (Domain Local), realiza o aninhamento seguro e cria uma pasta no File Server com as devidas permissões NTFS isoladas.

Onde rodar: Controlador de Domínio (Windows Server)

Import-Module ActiveDirectory

# Define os caminhos dinamicamente com base no domínio
$domainDN = (Get-ADDomain).DistinguishedName
$ouUsuarios = "OU=Usuarios,OU=Matriz,$domainDN"
$ouGrupos = "OU=Grupos,OU=Matriz,$domainDN"

# Define uma senha padrão segura
$senhaPadrao = ConvertTo-SecureString "SenhaFort3!@" -AsPlainText -Force

Write-Host "1. Criando Usuários..." -ForegroundColor Cyan
New-ADUser -Name "Joao RH" -SamAccountName "joao.rh" -AccountPassword $senhaPadrao -Enabled $true -Path $ouUsuarios
New-ADUser -Name "Maria RH" -SamAccountName "maria.rh" -AccountPassword $senhaPadrao -Enabled $true -Path $ouUsuarios
New-ADUser -Name "Pedro RH" -SamAccountName "pedro.rh" -AccountPassword $senhaPadrao -Enabled $true -Path $ouUsuarios
New-ADUser -Name "Lucas Fin" -SamAccountName "lucas.fin" -AccountPassword $senhaPadrao -Enabled $true -Path $ouUsuarios
New-ADUser -Name "Ana Fin" -SamAccountName "ana.fin" -AccountPassword $senhaPadrao -Enabled $true -Path $ouUsuarios

Write-Host "2. Criando Grupos e fazendo Aninhamento (AGDLP)..." -ForegroundColor Cyan
# Criação dos grupos (Cargo vs Acesso)
New-ADGroup -Name "GG_RH_Analistas" -GroupCategory Security -GroupScope Global -Path $ouGrupos
New-ADGroup -Name "DL_Pasta_RH_Leitura" -GroupCategory Security -GroupScope DomainLocal -Path $ouGrupos

# Aninhamento: Insere Usuários -> Grupo Global -> Grupo de Domínio Local
Add-ADGroupMember -Identity "GG_RH_Analistas" -Members "joao.rh", "maria.rh", "pedro.rh"
Add-ADGroupMember -Identity "DL_Pasta_RH_Leitura" -Members "GG_RH_Analistas"

Write-Host "3. Criando a Pasta e aplicando permissões NTFS..." -ForegroundColor Cyan
# Cria diretório alvo e limpa/aplica ACLs
New-Item -Path "C:\Dados_RH" -ItemType Directory -Force | Out-Null
$acl = Get-Acl "C:\Dados_RH"
$regraPermissao = New-Object System.Security.AccessControl.FileSystemAccessRule("DL_Pasta_RH_Leitura", "ReadAndExecute", "ContainerInherit, ObjectInherit", "None", "Allow")
$acl.AddAccessRule($regraPermissao)
Set-Acl -Path "C:\Dados_RH" -AclObject $acl

Write-Host "Cenário AGDLP concluído com sucesso!" -ForegroundColor Green



3. Configuração de Rede (Cliente)

Script utilizado nas estações de trabalho (Windows 10/11) para preparar a rede antes do ingresso no domínio. Ele detecta a interface de rede ativa e injeta IP estático, Subnet, Gateway e o DNS apontando obrigatoriamente para o IP do Controlador de Domínio.

Onde rodar: Estação de Trabalho (Windows 10 ou 11)
(⚠️ Altere as variáveis de IP conforme a rede do seu laboratório antes de executar)

# 1. Identifica a placa de rede ativa (Status = 'Up')
$adapter = Get-NetAdapter | Where-Object Status -eq 'Up'

# 2. Configura IP, Máscara e Gateway
New-NetIPAddress -InterfaceIndex $adapter.ifIndex -IPAddress 192.168.10.20 -PrefixLength 24 -DefaultGateway 192.168.10.1

# 3. Configura o DNS apontando para o Domain Controller
Set-DnsClientServerAddress -InterfaceIndex $adapter.ifIndex -ServerAddresses 192.168.10.10

Write-Host "Rede configurada com sucesso!" -ForegroundColor Green

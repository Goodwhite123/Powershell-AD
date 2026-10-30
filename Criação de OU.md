# Criação de Estrutura de OUs no Active Directory

Este documento fornece o script em PowerShell para automatizar a criação de uma estrutura padronizada de Unidades Organizacionais (OUs) num ambiente Active Directory.

## 📋 Estrutura de OUs

O script estabelece uma hierarquia, definindo a matriz e as respetivas sub-OUs para organização do domínio:
*   **Matriz** (OU Raiz na base do domínio)
    *   Usuarios
    *   Grupos
    *   Contas de Servico
    *   Estacoes de Trabalho

## 💻 Script PowerShell

```powershell
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
New-ADOrganizationalUnit -Name "Contas de Servico" -Path $matrizDN
New-ADOrganizationalUnit -Name "Estacoes de Trabalho" -Path $matrizDN

Write-Host "Estrutura de OUs criada com sucesso!" -ForegroundColor Green

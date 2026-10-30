# Criação de Estrutura de OUs no Active Directory

Este documento fornece o script em PowerShell para automatizar a criação de uma estrutura padronizada de Unidades Organizacionais (OUs) num ambiente Active Directory.

## 📋 Estrutura de OUs

O script estabelece uma hierarquia, definindo a matriz e as respetivas sub-OUs para organização do domínio[cite: 1]:
*   **Matriz** (OU Raiz na base do domínio)[cite: 1]
    *   Usuarios[cite: 1]
    *   Grupos[cite: 1]
    *   Contas de Servico[cite: 1]
    *   Estacoes de Trabalho[cite: 1]

## 💻 Script PowerShell

```powershell
# Importa o módulo do Active Directory[cite: 1]
Import-Module ActiveDirectory[cite: 1]

# Obtém o Distinguished Name (DN) do domínio atual (ex: DC=cyberlab,DC=local)[cite: 1]
$domainDN = (Get-ADDomain).DistinguishedName[cite: 1]

# 1. Cria a OU raiz "Matriz" na raiz do domínio[cite: 1]
New-ADOrganizationalUnit -Name "Matriz" -Path $domainDN[cite: 1]

# Define o caminho (Path) da nova OU "Matriz" para criar as sub-OUs dentro dela[cite: 1]
$matrizDN = "OU=Matriz,$domainDN"[cite: 1]

# 2. Cria as sub-OUs organizacionais dentro da OU "Matriz"[cite: 1]
New-ADOrganizationalUnit -Name "Usuarios" -Path $matrizDN[cite: 1]
New-ADOrganizationalUnit -Name "Grupos" -Path $matrizDN[cite: 1]
New-ADOrganizationalUnit -Name "Contas de Servico" -Path $matrizDN[cite: 1]
New-ADOrganizationalUnit -Name "Estacoes de Trabalho" -Path $matrizDN[cite: 1]

Write-Host "Estrutura de OUs criada com sucesso!" -ForegroundColor Green[cite: 1]

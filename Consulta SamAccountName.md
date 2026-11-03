# Consulta de Utilizadores no Active Directory

Este documento detalha o comando em PowerShell utilizado para pesquisar utilizadores específicos no Active Directory e formatar a saída para exibir apenas as informações essenciais.

## 📋 Detalhes do Comando

O script realiza uma pesquisa otimizada utilizando os seguintes componentes:
*   **Get-ADUser:** O cmdlet principal responsável por consultar a base de dados do Active Directory.
*   **-Filter "Name -like '*Joao*'":** Aplica um filtro de pesquisa. *Nota de melhoria: Adicionámos os asteriscos (`*`) como caracteres curinga (wildcards). Assim, o comando encontra qualquer utilizador que contenha "Joao" no nome (ex: "Joao Silva", "Maria Joao"), tornando a pesquisa muito mais eficiente.*
*   **Select-Object Name, SamAccountName:** Filtra as propriedades do resultado para não poluir o ecrã, exibindo apenas o Nome de exibição e o Nome de Logon de utilizador (SamAccountName).

## 💻 Script PowerShell

```powershell
# Importa o módulo do Active Directory (caso ainda não esteja carregado na sessão)
Import-Module ActiveDirectory

# Pesquisa utilizadores cujo nome contenha "Joao" e exibe o Nome e o SamAccountName
Get-ADUser -Filter "Name -like '*Debora*'" | Format-List Name, SamAccountName, UserPrincipalName

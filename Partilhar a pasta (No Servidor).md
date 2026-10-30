# Criação de Compartilhamento de Rede (SMB Share)

Este documento fornece o script em PowerShell para automatizar a criação de um compartilhamento de rede (SMB Share). A estratégia utilizada concede acesso total no nível do compartilhamento, delegando o controle de segurança real para as permissões NTFS previamente configuradas na pasta local.

## 📋 Detalhes do Compartilhamento

O script utiliza os seguintes parâmetros para criar a partilha:
*   **Nome do Compartilhamento (Name):** `Dados_RH` (Como a pasta será vista na rede).
*   **Caminho Local (Path):** `C:\Dados_RH` (O diretório físico no servidor/computador onde os arquivos estão armazenados).
*   **Permissão de Compartilhamento (FullAccess):** `Everyone` (Todos). *Nota: A segurança e restrição de acesso efetivas serão geridas pelas permissões NTFS da pasta `C:\Dados_RH`.*

## 💻 Script PowerShell

```powershell
# Cria a partilha de rede (SMB Share) e delega o controlo de segurança para as regras NTFS que já configurámos
New-SmbShare -Name "Dados_RH" -Path "C:\Dados_RH" -FullAccess "Everyone"

Write-Host "Compartilhamento 'Dados_RH' criado com sucesso!" -ForegroundColor Green

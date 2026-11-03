# Processo de Desligamento de Utilizadores (Offboarding no AD)

Este documento fornece o script em PowerShell para automatizar o processo de desligamento (*offboarding*) de colaboradores no Active Directory. O fluxo garante a segurança imediata do ambiente através do bloqueio de acesso, revogação de permissões, registo de auditoria e isolamento da conta.

## 📋 Etapas do Processo

O script executa o desligamento de forma segura e estruturada em 5 passos:
1.  **Validação de Existência (`try / catch`):** Verifica se a conta informada realmente existe no domínio antes de executar qualquer ação, interrompendo o script de forma segura caso o utilizador não seja encontrado.
2.  **Bloqueio Imediato (`Disable-ADAccount`):** Desativa a conta do utilizador para interromper imediatamente qualquer sessão ativa ou tentativa de novo logon.
3.  **Revogação de Acessos (`Remove-ADGroupMember`):** Remove o utilizador de todos os grupos de segurança e distribuição (mantendo apenas o grupo obrigatório `Domain Users`), garantindo a revogação de acessos a pastas, sistemas e políticas.
4.  **Auditoria e Higiene de Dados (`Set-ADUser`):** Regista na descrição da conta a data, a hora e o administrador responsável pela execução (`$env:USERNAME`), além de limpar os campos de Gestor, Cargo e Departamento.
5.  **Isolamento Organizacional (`Move-ADObject`):** Move o objeto do utilizador para uma Unidade Organizacional (OU) dedicada a contas inativas (`Desligados`), mantendo a estrutura do AD limpa e evitando a aplicação desnecessária de GPOs.

## 💻 Script PowerShell

```powershell
# Importa o módulo do Active Directory
Import-Module ActiveDirectory

# 1. Defina o login técnico (SamAccountName) do utilizador a desligar
$Utilizador = "Pipoca"

try {
    $Conta = Get-ADUser -Identity $Utilizador -ErrorAction Stop
} catch {
    Write-Host "ERRO: O utilizador '$Utilizador' não foi encontrado!" -ForegroundColor Red
    return
}

Write-Host "A iniciar o processo de desligamento para: $($Conta.Name)..." -ForegroundColor Cyan

# 2. Desativar a conta
Disable-ADAccount -Identity $Utilizador
Write-Host "[OK] Conta desativada." -ForegroundColor Green

# 3. Remover de todos os grupos (exceto Domain Users)
$GruposAtuais = Get-ADPrincipalGroupMembership -Identity $Utilizador | Where-Object { $_.Name -ne "Domain Users" }

foreach ($Grupo in $GruposAtuais) {
    Remove-ADGroupMember -Identity $Grupo -Members $Utilizador -Confirm:$false
    Write-Host "[REVOGADO] Removido do grupo: $($Grupo.Name)" -ForegroundColor Yellow
}

# 4. Limpar metadados (higiene de licenças) e registar auditoria
$DataHora = Get-Date -Format "dd/MM/yyyy HH:mm"
$AdminExecutor = $env:USERNAME

Set-ADUser -Identity $Utilizador `
    -Description "DESLIGADO a $DataHora por$AdminExecutor" `
    -Clear Manager, Title, Department

Write-Host "[OK] Metadados limpos e auditoria registada." -ForegroundColor Green

# 5. Mover para a OU "Desligados" (Caminho exato da sua infraestrutura)
$OU_Destino = "OU=Desligados,OU=Matriz,DC=cyberlab,DC=local"

Move-ADObject -Identity $Conta.DistinguishedName -TargetPath $OU_Destino
Write-Host "[OK] Utilizador movido para a OU: $OU_Destino" -ForegroundColor Green
Write-Host "Processo concluído com sucesso!" -ForegroundColor Cyan
```

## ⚙️ Pré-requisitos e Instruções de Execução

*   **Privilégios:** O operador deve possuir permissões de **Administrador de Domínio (Domain Admin)** ou delegação específica para desativar contas, alterar grupos, editar atributos e mover objetos entre OUs.
*   **Estrutura Prévia:** A Unidade Organizacional de destino (`OU=Desligados,OU=Matriz,DC=cyberlab,DC=local`) já deve existir no Active Directory antes da execução do script.
*   **Personalização:** Altere o valor da variável `$Utilizador` para o login (`SamAccountName`) do colaborador a ser desligado e ajuste a variável `$OU_Destino` caso o caminho do seu domínio seja diferente.

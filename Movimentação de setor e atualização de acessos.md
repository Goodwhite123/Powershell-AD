# Movimentação de Setor e Atualização de Acessos (AD)

Este documento fornece o script em PowerShell para automatizar a transferência de departamento de um usuário no Active Directory. O processo aplica o **Princípio do Menor Privilégio**, revogando todos os acessos antigos antes de conceder as permissões do novo cargo (evitando o acúmulo indevido de permissões, conhecido como *Privilege Creep*).

## 📋 Etapas do Processo

O script executa a transição do colaborador em 4 passos automatizados:
1.  **Auditoria de Grupos (`Get-ADPrincipalGroupMembership`):** Lista todos os grupos aos quais o usuário pertence atualmente, ignorando o grupo primário padrão (`Domain Users`), que não pode nem deve ser removido.
2.  **Revogação Limpa (`Remove-ADGroupMember`):** Percorre a lista encontrada e remove o usuário de cada grupo antigo sem exigir confirmação manual na tela (`-Confirm:$false`).
3.  **Concessão de Novo Acesso (`Add-ADGroupMember`):** Insere o usuário no grupo de segurança correspondente à sua nova função (`GG_Financeiro_Analistas`).
4.  **Atualização de Metadados (`Set-ADUser`):** Atualiza os atributos organizacionais do perfil do usuário (Departamento e Cargo) no catálogo do Active Directory.

## 💻 Script PowerShell

```powershell
# Importa o módulo do Active Directory
Import-Module ActiveDirectory

# Define o usuário que será movimentado (SamAccountName)
$Usuario = "joao.silva"

# 1. Busca TODOS os grupos atuais do usuário (exceto o grupo padrão Domain Users)
$GruposAtuais = Get-ADPrincipalGroupMembership -Identity $Usuario \vert{} Where-Object {$_.Name -ne "Domain Users" }

# 2. Loop que passa grupo por grupo removendo o usuário automaticamente
foreach ($Grupo in$GruposAtuais) {
    Remove-ADGroupMember -Identity $Grupo -Members $Usuario -Confirm:$false
    Write-Host "Acesso revogado do grupo: $($Grupo.Name)" -ForegroundColor Yellow
}

# 3. Adiciona no grupo do novo setor (Concessão limpa)
Add-ADGroupMember -Identity "GG_Financeiro_Analistas" -Members $Usuario
Write-Host "Acesso concedido ao grupo: GG_Financeiro_Analistas" -ForegroundColor Green

# 4. Atualiza os metadados da conta
Set-ADUser -Identity $Usuario -Department "Financeiro" -Title "Analista Financeiro"
Write-Host "Dados organizacionais de '$Usuario' atualizados com sucesso!" -ForegroundColor Cyan

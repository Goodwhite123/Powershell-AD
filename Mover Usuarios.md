$Usuario = "Emerson Silva"

# 1. Remove do grupo antigo (Revogação de acesso)
Remove-ADGroupMember -Identity "GG-FINANCEIRO" -Members $Usuario -Confirm:$false

# 2. Adiciona no grupo novo (Concessão de acesso)
Add-ADGroupMember -Identity "GG_RH_Analistas" -Members $Usuario

# 3. Atualiza os metadados da conta para refletir a nova realidade
Set-ADUser -Identity $Usuario -Department "RH" -Title "Hunter Talents"

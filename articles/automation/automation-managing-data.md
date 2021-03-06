<properties 
   pageTitle="Gerenciando os dados da Automação do Azure | Microsoft Azure"
   description="Este artigo contém vários tópicos sobre o gerenciamento de um ambiente da Automação do Azure. Atualmente, inclui a Retenção de dados e o backup da Recuperação de desastres na Automação do Azure."
   services="automation"
   documentationCenter=""
   authors="SnehaGunda"
   manager="stevenka"
   editor="tysonn" />
<tags 
   ms.service="automation"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="05/02/2016"
   ms.author="bwren;sngun" />

# Gerenciando dados da Automação do Azure

Este artigo contém vários tópicos sobre o gerenciamento de um ambiente da Automação do Azure.

## Retenção de dados

Quando você exclui um recurso na Automação do Azure, ele é retido por 90 dias para fins de auditoria antes de ser removido permanentemente. Você não pode ver nem usar o recurso durante esse período. Essa política também se aplica a recursos que pertencem a uma conta de automação que é excluída.

A Automação do Azure exclui de forma automática e remove de forma permanente os trabalhos com mais de 90 dias.

A tabela a seguir resume a política de retenção para diferentes recursos.

|Dados|Política|
|:---|:---|
|Contas|Removidas permanentemente 90 dias depois que a conta é excluída por um usuário.|
|Ativos|Removidos permanentemente 90 dias depois que o ativo é excluído por um usuário ou 90 dias depois que a conta que contém o ativo é excluída por um usuário.|
|Módulos|Removidos permanentemente 90 dias depois que o módulo é excluído por um usuário ou 90 dias depois que a conta que contém o módulo é excluída por um usuário.|
|Runbooks|Removidos permanentemente 90 dias depois que o recurso é excluído por um usuário ou 90 dias depois que a conta que contém o recurso é excluída por um usuário.|
|Trabalhos|Excluído e removidos permanentemente 90 dias após a última modificação. Isso pode ocorrer depois que o trabalho é concluído, interrompido ou suspenso.|
|Arquivos de configurações/MOF de nó| A configuração de nó antigo é removida permanentemente 90 dias depois que uma nova configuração de nó é gerada.|
|Nós DSC| Removido de forma permanente 90 dias após o nó é cancelar o registro da conta de automação usando o portal do Azure ou o [AzureRMAutomationDscNode Unregister](https://msdn.microsoft.com/library/mt603500.aspx) cmdlet do Windows PowerShell. Nós também permanentemente são removidos de 90 dias depois que a conta que contém que o nó é excluída por um usuário. |
|Relatórios de Nó| Removido de forma permanente 90 dias depois que um novo relatório é gerado para esse nó|

A política de retenção se aplica a todos os usuários e, atualmente não, pode ser personalizada.

## Fazendo backup da Automação do Azure

Quando você exclui uma conta de automação no Microsoft Azure, todos os objetos na conta são excluídos, incluindo runbooks, módulos, configurações, trabalhos e ativos. Os objetos não podem ser recuperados depois que a conta é excluída. Você pode usar as informações a seguir para fazer backup do conteúdo de sua conta de automação antes de excluí-la.

### Runbooks

Você pode exportar seus runbooks para arquivos de script usando o Portal de Gerenciamento do Azure ou o cmdlet [Get-AzureAutomationRunbookDefinition](https://msdn.microsoft.com/library/dn690269.aspx) no Windows PowerShell. Esses arquivos de script podem ser importados para outra conta de automação, conforme discutido em [Criando ou importando um runbook](https://msdn.microsoft.com/library/dn643637.aspx).


### Módulos de integração

Você não pode exportar módulos de integração da Automação do Azure. Você deve garantir que eles estejam disponíveis fora da conta de automação.

### Ativos

Não é possível exportar [ativos](https://msdn.microsoft.com/library/dn939988.aspx) da Automação do Azure. Usando o Portal de Gerenciamento do Azure, você deve observar os detalhes de variáveis, credenciais, certificados, conexões e agendas. Você deve criar manualmente qualquer ativo que seja usado por runbooks importados para outra automação.

Você pode usar [cmdlets do Azure](https://msdn.microsoft.com/library/dn690262.aspx) para recuperar os detalhes de ativos não criptografados e salvá-los para referência futura ou criar ativos equivalentes em outra conta de automação.

Não é possível recuperar o valor de variáveis criptografadas nem o campo de senha de credenciais usando cmdlets. Se não souber esses valores, você poderá recuperá-los de um runbook usando as atividades [Get-AutomationVariable](https://msdn.microsoft.com/library/dn940012.aspx) e [Get-AutomationPSCredential](https://msdn.microsoft.com/library/dn940015.aspx).

Você não pode exportar certificados da Automação do Azure. Você deve garantir que todos os certificados estejam disponíveis fora do Azure.

### Configurações DSC

Você pode exportar suas configurações para arquivos de script usando o Portal de Gerenciamento do Azure ou o cmdlet [Export-AzureRmAutomationDscConfiguration](https://msdn.microsoft.com/library/mt603485.aspx) no Windows PowerShell. Essas configurações podem ser importadas e usadas em outra conta de automação.


##Replicação geográfica na Automação do Azure

A replicação geográfica, padrão em contas da Automação do Azure, faz o backup de dados da conta em uma região geográfica diferente para obter redundância. Você pode escolher uma região primária durante a configuração de sua conta e, em seguida, uma região secundária é atribuída automaticamente a ela. Os dados secundários, copiados da região primária, são atualizados continuamente em caso de perda de dados.

A tabela a seguir mostra os emparelhamentos disponíveis das regiões primárias e secundárias.

|Primário |Secundário
| ---------------   |----------------
|Centro-Sul dos Estados Unidos |Centro-Norte dos EUA
|Leste dos EUA 2 |Centro dos EUA
|Europa Ocidental |Norte da Europa
|Sudeste da Ásia |Ásia Oriental
|Leste do Japão |Oeste do Japão

No evento improvável de os dados de uma região primária serem perdidos, a Microsoft tentará recuperá-los. Se não for possível recuperar os dados primários, o failover geográfico será executado e os clientes afetados receberão uma notificação em suas assinaturas.

<!---HONumber=AcomDC_0511_2016-->
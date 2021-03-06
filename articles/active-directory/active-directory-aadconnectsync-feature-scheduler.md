<properties
   pageTitle="Sincronização do Azure AD Connect: Agendador | Microsoft Azure"
   description="Este tópico descreve o recurso Agendador interno na sincronização do Azure AD Connect."
   services="active-directory"
   documentationCenter=""
   authors="AndKjell"
   manager="femila"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="08/04/2016"
   ms.author="andkjell"/>

# Sincronização do Azure AD Connect: agendador
Este tópico descreve o agendador interno na Sincronização do Azure AD Connect (também conhecido como mecanismo de sincronização).

Esse recurso foi introduzido com a compilação 1.1.105.0 (lançada em fevereiro de 2016).

## Visão geral
A sincronização do Azure AD Connect sincronizará mudanças ocorridas em seu diretório local usando um agendador. Há dois processos de agendador, um para sincronização de senha e outro para sincronização de atributo/objeto, e tarefas de manutenção. Este tópico abordará as tarefas.

Em versões anteriores, o agendador de objetos e atributos ficava fora do mecanismo de sincronização e o agendador de tarefas do Windows ou um serviço separado do Windows era usado para disparar o processo de sincronização. O agendador faz parte dos recursos internos nas versões 1.1 do mecanismo de sincronização e permite alguma personalização. A frequência de sincronização do novo padrão é de 30 minutos.

O agendador é responsável por duas tarefas:

- **Ciclo de sincronização**. O processo de importação, sincronização e exportação das alterações.
- **Tarefas de manutenção**. Renove as chaves e certificados para a redefinição de Senha e o DRS (Serviço de Registro de Dispositivo). Limpe as entradas antigas no log de operações.

O agendador em si está sempre em execução, mas ele pode ser configurado para executar apenas uma ou nenhuma dessas tarefas. Por exemplo, se você precisar ter seu próprio processo de ciclo de sincronização, poderá desabilitar essa tarefa no agendador, mas ainda executar a tarefa de manutenção.

## Configuração do agendador
Para ver as configurações atuais, acesse o PowerShell e execute `Get-ADSyncScheduler`. Ele mostrará algo assim:

![GetSyncScheduler](./media/active-directory-aadconnectsync-feature-scheduler/getsynccyclesettings.png)

Se você vir **O comando de sincronização ou o cmdlet não está disponível** quando executar esse cmdlet, o módulo do PowerShell não estará carregado. Isso poderá ocorrer se você executar o Azure AD Connect em um controlador de domínio ou em um servidor com níveis mais altos de restrição do PowerShell do que as configurações padrão. Se você vir esse erro, execute `Import-Module ADSync` para disponibilizar o cmdlet.

- **AllowedSyncCycleInterval**. O AD do Azure com maior frequência permitirá a realização das sincronizações. Você não pode sincronizar com maior frequência do que isso e ainda ter suporte.
- **CurrentlyEffectiveSyncCycleInterval**. O agendamento atualmente em vigor. Ele terá o mesmo valor de CustomizedSyncInterval (se definido) se não for mais frequente do que AllowedSyncInterval. Se você alterar CustomizedSyncCycleInterval, isso entrará em vigor após o próximo ciclo de sincronização.
- **CustomizedSyncCycleInterval**. Se você quiser que o agendador seja executado em intervalos diferentes do padrão de 30 minutos, terá que definir nessa configuração. Na figura acima, o agendador foi definido para ser executado a cada hora. Se você definir isso como um valor menor do que AllowedSyncInterval, a última opção será usada.
- **NextSyncCyclePolicyType**. Delta ou Inicial. Define se a próxima execução deve processar somente alterações delta ou se deve fazer uma importação e uma sincronização completas, o que também reprocessaria as regras novas ou alteradas.
- **NextSyncCycleStartTimeInUTC**. Da próxima vez, o agendador iniciará o próximo ciclo de sincronização.
- **PurgeRunHistoryInterval**. Os logs de operação de tempo devem ser mantidos. Eles podem ser analisados no gerenciador do serviço de sincronização. O padrão é mantê-los por sete dias.
- **SyncCycleEnabled**. Indica se o agendador está executando os processos de exportação, importação e sincronização como parte de sua operação.
- **MaintenanceEnabled**. Mostra se o processo de manutenção está habilitado. Ele atualizará os certificados/chaves e limpará o log de operações.
- **IsStagingModeEnabled**. Mostra se o [modo de preparo](active-directory-aadconnectsync-operations.md#staging-mode) está habilitado.

Você pode alterar algumas dessas configurações com `Set-ADSyncScheduler`. Os parâmetros a seguir podem ser modificados:

- CustomizedSyncCycleInterval
- NextSyncCyclePolicyType
- PurgeRunHistoryInterval
- SyncCycleEnabled
- MaintenanceEnabled

A configuração do agendador é armazenada no Azure AD. Se tiver um servidor de preparo, qualquer alteração no servidor primário também afetará o servidor de preparo (com exceção de IsStagingModeEnabled).

### CustomizedSyncCycleInterval
Sintaxe: `Set-ADSyncScheduler -CustomizedSyncCycleInterval d.HH:mm:ss` d - dias, HH - horas, mm - minutos, ss - segundos

Exemplo: `Set-ADSyncScheduler -CustomizedSyncCycleInterval 03:00:00` alterará o Agendador para executar a cada três horas.

Exemplo: `Set-ADSyncScheduler -CustomizedSyncCycleInterval 1.0:0:0` alterará o Agendador para executar diariamente.

## Iniciar o agendador
Por padrão, o agendador será executado a cada 30 minutos. Em alguns casos, é bom executar um ciclo de sincronização entre os ciclos agendados ou terá que executar um tipo diferente.

**Ciclo de sincronização delta** Um ciclo de sincronização delta inclui as seguintes etapas:

- Importação delta em todos os conectores
- Sincronização delta em todos os conectores
- Exportação em todos os conectores

É possível que você tenha uma alteração urgente que deva ser sincronizada imediatamente e precise executar um ciclo manualmente. Se precisar executar um ciclo manualmente, execute `Start-ADSyncSyncCycle -PolicyType Delta` no PowerShell.

**Ciclo de sincronização completo** Caso tenha feito uma das alterações de configuração a seguir, será necessário executar um ciclo de sincronização completo (também conhecido como Inicial):

- Adicionou mais objetos ou atributos para serem importados de um diretório de origem
- Realizou alterações nas regras de sincronização
- Alterou a [filtragem](active-directory-aadconnectsync-configure-filtering.md) para incluir um número diferente de objetos

Se você tiver realizado uma dessas alterações, precisará executar um ciclo de sincronização completa para que o mecanismo de sincronização possa reconsolidar os espaços conectores. Um ciclo de sincronização completa inclui as seguintes etapas:

- Importação completa de todos os conectores
- Sincronização completa de todos os conectores
- Exportação em todos os conectores

Para iniciar um ciclo de sincronização completo, execute `Start-ADSyncSyncCycle -PolicyType Initial` em um prompt do PowerShell. Isso iniciará um ciclo completo de sincronização.

## Parar o agendador
Se o agendador estiver executando um ciclo de sincronização, talvez seja necessário interrompê-lo. Por exemplo, se você iniciar o assistente de instalação e receber este erro:

![SyncCycleRunningError](./media/active-directory-aadconnectsync-feature-scheduler/synccyclerunningerror.png)

Se um ciclo de sincronização estiver em execução, você não poderá alterar a configuração. Você pode aguardar até que o agendador conclua o processo ou pode interrompê-lo para realizar suas alterações logo em seguida. Parar o ciclo atual não é prejudicial e as alterações ainda não processadas serão processadas na próxima execução.

1. Comece informando o agendador para interromper o ciclo atual com o cmdlet `Stop-ADSyncSyncCycle` do PowerShell.
2. Parar o agendador não interromperá a tarefa atual do conector atual. Para forçar a interrupção do Conector, execute as seguintes ações: ![StopAConnector](./media/active-directory-aadconnectsync-feature-scheduler/stopaconnector.png)
    - Inicie o **Serviço de Sincronização** no menu Iniciar. Vá para **Conectores**, realce o Conector com o estado **Executando** e selecione **Parar** em Ações.

O agendador ainda está ativo e será iniciado novamente na próxima oportunidade.

## Agendador personalizado
Os cmdlets documentados nesta seção só estão disponíveis no build [1\.1.130.0](active-directory-aadconnect-version-history.md#111300) e posteriores.

Se o agendador interno não atender às suas necessidades, você poderá fazer o agendamento dos Conectores usando o PowerShell.

### Invoke-ADSyncRunProfile
Você pode iniciar um perfil para um Conector desta forma:

```
Invoke-ADSyncRunProfile -ConnectorName "name of connector" -RunProfileName "name of profile"
```

Os nomes a serem usados como [Nomes de conector](active-directory-aadconnectsync-service-manager-ui-connectors.md) e [Nomes de Perfil de Execução](active-directory-aadconnectsync-service-manager-ui-connectors.md#configure-run-profiles) podem ser encontrados na [Interface do Usuário do Synchronization Service Manager](active-directory-aadconnectsync-service-manager-ui.md).

![Invocar perfil de execução](./media/active-directory-aadconnectsync-feature-scheduler/invokerunprofile.png)

O cmdlet `Invoke-ADSyncRunProfile` é síncrono, ou seja, ele não retornará o controle até que o Conector tenha concluído a operação, seja com êxito ou com erro.

Quando você agendar seus Conectores, a recomendação é agendá-los na seguinte ordem:

1. (Completo/Delta) Importar de diretórios locais, como o Active Directory
2. (Completo/Delta) Importar do Azure AD
3. (Completo/Delta) Sincronizar de diretórios locais, como o Active Directory
4. (Completo/Delta) Sincronização do Azure AD
5. Exportar para o Azure AD
6. Exportar para diretórios locais, como o Active Directory

Se você examinar o agendador interno, esta será a ordem em que os Conectores serão executados.

### Get-ADSyncConnectorRunStatus
Você também pode monitorar o mecanismo de sincronização para ver se ele está ocupado ou ocioso. Esse cmdlet retornará um resultado vazio se o mecanismo de sincronização estiver ocioso e não estiver executando um Conector. Se um Conector estiver em execução, ele retornará o nome do Conector.

```
Get-ADSyncConnectorRunStatus
```

![Status de execução do conector](./media/active-directory-aadconnectsync-feature-scheduler/getconnectorrunstatus.png) Na imagem acima, a primeira linha é de um estado em que o mecanismo de sincronização está ocioso. A segunda linha é de quando o Conector do Azure AD está em execução.

## Agendador e o assistente de instalação
Se você iniciar o assistente de instalação, o agendador será temporariamente suspenso. Isso ocorre porque pressupõe-se que você fará alterações na configuração e elas não podem ser aplicadas se o mecanismo de sincronização estiver ativamente em execução. Por esse motivo, não deixe o assistente de instalação aberto, já que ele impedirá que o mecanismo de sincronização execute ações de sincronização.

## Próximas etapas
Saiba mais sobre a configuração de [sincronização do Azure AD Connect](active-directory-aadconnectsync-whatis.md).

Saiba mais sobre [Como integrar suas identidades locais ao Active Directory do Azure](active-directory-aadconnect.md).

<!---HONumber=AcomDC_0928_2016-->
<properties
	pageTitle="Métricas do Azure Monitor - métricas suportadas por tipo de recurso | Microsoft Azure"
	description="Lista de métricas disponíveis para cada tipo de recurso com o Azure Monitor."
	authors="johnkemnetz"
	manager="rboucher"
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>

<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/26/2016"
	ms.author="johnkem"/>

# Métricas compatíveis com o Azure Monitor

O Azure Monitor fornece várias maneiras de interagir com as métricas, incluindo a criação de gráficos para os mesmos no portal, acessá-las por meio da API REST ou consultá-las usando o PowerShell ou a CLI. Abaixo está uma lista completa de todas as métricas atualmente disponíveis do pipeline de métrica do Azure Monitor.

> [AZURE.NOTE] Outras métricas podem estar disponíveis no portal ou usando as APIs herdadas. Essa lista inclui apenas as métricas de visualização pública disponíveis usando a visualização pública do pipeline de métrica consolidado do Azure Monitor.

## Microsoft.Batch/batchAccounts

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|CoreCount|Contagem de núcleos|Porcentagem|Média|Número médio de núcleos na conta do Lote|
|TotalNodeCount|Contagem de nós|Porcentagem|Média|Número médio de nós na conta do lote|
|CreatingNodeCount|Criação de contagem de nós|Porcentagem|Média|Número de nós sendo criados|
|StartingNodeCount|Contagem inicial dos nós|Porcentagem|Média|Número de nós iniciais|
|WaitingForStartTaskNodeCount|Contagem de nós para tarefa de inicialização em espera|Porcentagem|Média|Número de nós aguardando a conclusão da Tarefa de Inicialização|
|StartTaskFailedNodeCount|Falha na contagem de nós para a tarefa de inicialização|Porcentagem|Média|Número de nós com falha na tarefa de inicialização|
|IdleNodeCount|Contagem de nós ociosos|Porcentagem|Média|Número de nós ociosos|
|OfflineNodeCount|Contagem de nós off-line|Porcentagem|Média|Número de nós off-line|
|RebootingNodeCount|Contagem de nós de reinicialização|Porcentagem|Média|Número de nós de reinicialização|
|ReimagingNodeCount|Contagem de nós refazendo a imagem|Porcentagem|Média|Número de nós refazendo a imagem|
|RunningNodeCount|Contagem de nós em execução|Porcentagem|Média|Número de nós em execução|
|LeavingPoolNodeCount|Contagem de nós saindo do pool|Porcentagem|Média|Número de nós saindo do pool|
|UnusableNodeCount|Contagem de nós inutilizáveis|Porcentagem|Média|Número de nós inutilizáveis|
|TaskStartEvent|Eventos da tarefa de inicialização|Contagem|Total|Número total de tarefas que iniciaram|
|TaskCompleteEvent|Eventos de conclusão de tarefa|Contagem|Total|Número total de tarefas concluídas|
|TaskFailEvent|Eventos de falha de tarefa|Contagem|Total|Número total de tarefas que foram concluídas em um estado com falha|
|PoolCreateEvent|Eventos de criação de pool|Contagem|Total|Número total de pools criados|
|PoolResizeStartEvent|Eventos de início de redimensionamento de pool|Contagem|Total|Número total de tarefas de redimensionamento de pool que iniciaram|
|PoolResizeCompleteEvent|Eventos de conclusão de redimensionamento de pool|Contagem|Total|Número total de redimensionamentos de pool concluídos|
|PoolDeleteStartEvent|Eventos de início de exclusão de pool|Contagem|Total|Número total de exclusões de pool iniciados|
|PoolDeleteCompleteEvent|Eventos de conclusão de exclusão do pool|Contagem|Total|Número total de exclusões de pool concluídas|

## Microsoft.Cache/redis

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|connectedclients|Clientes conectados|Contagem|Máximo||
|totalcommandsprocessed|Total de Operações|Contagem|Total||
|cachehits|Acertos do Cache|Contagem|Total||
|cachemisses|Erros de Cache|Contagem|Total||
|getcommands|Gets|Contagem|Total||
|setcommands|Sets|Contagem|Total||
|evictedkeys|Chaves removidas|Contagem|Total||
|totalkeys|Total de chaves|Contagem|Máximo||
|expiredkeys|Chaves expiradas|Contagem|Total||
|usedmemory|Memória Usada|Bytes|Máximo||
|usedmemoryRss|Memória RSS Usada|Bytes|Máximo||
|serverLoad|Carga do Servidor|Porcentagem|Máximo||
|cacheWrite|Gravação no Cache|BytesPerSecond|Máximo||
|cacheRead|Cache Lido|BytesPerSecond|Máximo||
|percentProcessorTime|CPU|Porcentagem|Máximo||
|connectedclients0|Clientes conectados (Fragmento 0)|Contagem|Máximo||
|totalcommandsprocessed0|Total de operações (Fragmento 0)|Contagem|Total||
|cachehits0|Ocorrências no cache (Fragmento 0)|Contagem|Total||
|cachemisses0|Perdas no cache (Fragmento 0)|Contagem|Total||
|getcommands0|Gets (Fragmento 0)|Contagem|Total||
|setcommands0|Sets (Fragmento 0)|Contagem|Total||
|evictedkeys0|Chaves removidas (Fragmento 0)|Contagem|Total||
|totalkeys0|Total de chaves (Nó 0)|Contagem|Máximo||
|expiredkeys0|Chaves expiradas (Fragmento 0)|Contagem|Total||
|usedmemory0|Memória usada (Fragmento 0)|Bytes|Máximo||
|usedmemoryRss0|Memória RSS usada (Fragmento 0)|Bytes|Máximo||
|serverLoad0|Carga do servidor (Fragmento 0)|Porcentagem|Máximo||
|cacheWrite0|Gravação no cache (Fragmento 0)|BytesPerSecond|Máximo||
|cacheRead0|Leitura no cache (Fragmento 0)|BytesPerSecond|Máximo||
|percentProcessorTime0|CPU (Fragmento 0)|Porcentagem|Máximo||
|connectedclients1|Clientes conectados (Fragmento 1)|Contagem|Máximo||
|totalcommandsprocessed1|Total de operações (Fragmento 1)|Contagem|Total||
|cachehits1|Ocorrências no cache (Fragmento 1)|Contagem|Total||
|cachemisses1|Perdas no cache (Fragmento 1)|Contagem|Total||
|getcommands1|Gets (Fragmento 1)|Contagem|Total||
|setcommands1|Sets (Fragmento 1)|Contagem|Total||
|evictedkeys1|Chaves removidas (Fragmento 1)|Contagem|Total||
|totalkeys1|Total de chaves (Nó 1)|Contagem|Máximo||
|expiredkeys1|Chaves expiradas (Fragmento 1)|Contagem|Total||
|usedmemory1|Memória usada (Fragmento 1)|Bytes|Máximo||
|usedmemoryRss1|Memória RSS usada (Fragmento 1)|Bytes|Máximo||
|serverLoad1|Carga do servidor (Fragmento 1)|Porcentagem|Máximo||
|cacheWrite1|Gravação no cache (Fragmento 1)|BytesPerSecond|Máximo||
|cacheRead1|Leitura no cache (Fragmento 1)|BytesPerSecond|Máximo||
|percentProcessorTime1|CPU (Fragmento 1)|Porcentagem|Máximo||
|connectedclients2|Clientes conectados (Fragmento 2)|Contagem|Máximo||
|totalcommandsprocessed2|Total de operações (Fragmento 2)|Contagem|Total||
|cachehits2|Ocorrências no cache (Fragmento 2)|Contagem|Total||
|cachemisses2|Perdas no cache (Fragmento 2)|Contagem|Total||
|getcommands2|Gets (Fragmento 2)|Contagem|Total||
|setcommands2|Sets (Fragmento 2)|Contagem|Total||
|evictedkeys2|Chaves removidas (Fragmento 2)|Contagem|Total||
|totalkeys2|Total de chaves (Nó 2)|Contagem|Máximo||
|expiredkeys2|Chaves expiradas (Fragmento 2)|Contagem|Total||
|usedmemory2|Memória usada (Fragmento 2)|Bytes|Máximo||
|usedmemoryRss2|Memória RSS usada (Fragmento 2)|Bytes|Máximo||
|serverLoad2|Carga do servidor (Fragmento 2)|Porcentagem|Máximo||
|cacheWrite2|Gravação no cache (Fragmento 2)|BytesPerSecond|Máximo||
|cacheRead2|Leitura no cache (Fragmento 2)|BytesPerSecond|Máximo||
|percentProcessorTime2|CPU (Fragmento 2)|Porcentagem|Máximo||
|connectedclients3|Clientes conectados (Fragmento 3)|Contagem|Máximo||
|totalcommandsprocessed3|Total de operações (Fragmento 3)|Contagem|Total||
|cachehits3|Ocorrências no cache (Fragmento 3)|Contagem|Total||
|cachemisses3|Perdas no cache (Fragmento 3)|Contagem|Total||
|getcommands3|Gets (Fragmento 3)|Contagem|Total||
|setcommands3|Sets (Fragmento 3)|Contagem|Total||
|evictedkeys3|Chaves removidas (Fragmento 3)|Contagem|Total||
|totalkeys3|Total de chaves (Nó 3)|Contagem|Máximo||
|expiredkeys3|Chaves expiradas (Fragmento 3)|Contagem|Total||
|usedmemory3|Memória usada (Fragmento 3)|Bytes|Máximo||
|usedmemoryRss3|Memória RSS usada (Fragmento 3)|Bytes|Máximo||
|serverLoad3|Carga do servidor (Fragmento 3)|Porcentagem|Máximo||
|cacheWrite3|Gravação no cache (Fragmento 3)|BytesPerSecond|Máximo||
|cacheRead3|Leitura no cache (Fragmento 3)|BytesPerSecond|Máximo||
|percentProcessorTime3|CPU (Fragmento 3)|Porcentagem|Máximo||
|connectedclients4|Clientes conectados (Fragmento 4)|Contagem|Máximo||
|totalcommandsprocessed4|Total de operações (Fragmento 4)|Contagem|Total||
|cachehits4|Ocorrências no cache (Fragmento 4)|Contagem|Total||
|cachemisses4|Perdas no cache (Fragmento 4)|Contagem|Total||
|getcommands4|Gets (Fragmento 4)|Contagem|Total||
|setcommands4|Sets (Fragmento 4)|Contagem|Total||
|evictedkeys4|Chaves removidas (Fragmento 4)|Contagem|Total||
|totalkeys4|Total de chaves (Nó 4)|Contagem|Máximo||
|expiredkeys4|Chaves expiradas (Fragmento 4)|Contagem|Total||
|usedmemory4|Memória usada (Fragmento 4)|Bytes|Máximo||
|usedmemoryRss4|Memória RSS usada (Fragmento 4)|Bytes|Máximo||
|serverLoad4|Carga do servidor (Fragmento 4)|Porcentagem|Máximo||
|cacheWrite4|Gravação no cache (Fragmento 4)|BytesPerSecond|Máximo||
|cacheRead4|Leitura no cache (Fragmento 4)|BytesPerSecond|Máximo||
|percentProcessorTime4|CPU (Fragmento 4)|Porcentagem|Máximo||
|connectedclients5|Clientes conectados (Fragmento 5)|Contagem|Máximo||
|totalcommandsprocessed5|Total de operações (Fragmento 5)|Contagem|Total||
|cachehits5|Ocorrências no cache (Fragmento 5)|Contagem|Total||
|cachemisses5|Perdas no cache (Fragmento 5)|Contagem|Total||
|getcommands5|Gets (Fragmento 5)|Contagem|Total||
|setcommands5|Sets (Fragmento 5)|Contagem|Total||
|evictedkeys5|Chaves removidas (Fragmento 5)|Contagem|Total||
|totalkeys5|Total de chaves (Nó 5)|Contagem|Máximo||
|expiredkeys5|Chaves expiradas (Fragmento 5)|Contagem|Total||
|usedmemory5|Memória usada (Fragmento 5)|Bytes|Máximo||
|usedmemoryRss5|Memória RSS usada (Fragmento 5)|Bytes|Máximo||
|serverLoad5|Carga do servidor (Fragmento 5)|Porcentagem|Máximo||
|cacheWrite5|Gravação no cache (Fragmento 5)|BytesPerSecond|Máximo||
|cacheRead5|Leitura no cache (Fragmento 5)|BytesPerSecond|Máximo||
|percentProcessorTime5|CPU (Fragmento 5)|Porcentagem|Máximo||
|connectedclients6|Clientes conectados (Fragmento 6)|Contagem|Máximo||
|totalcommandsprocessed6|Total de operações (Fragmento 6)|Contagem|Total||
|cachehits6|Ocorrências no cache (Fragmento 6)|Contagem|Total||
|cachemisses6|Perdas no cache (Fragmento 6)|Contagem|Total||
|getcommands6|Gets (Fragmento 6)|Contagem|Total||
|setcommands6|Sets (Fragmento 6)|Contagem|Total||
|evictedkeys6|Chaves removidas (Fragmento 6)|Contagem|Total||
|totalkeys6|Total de chaves (Nó 6)|Contagem|Máximo||
|expiredkeys6|Chaves expiradas (Fragmento 6)|Contagem|Total||
|usedmemory6|Memória usada (Fragmento 6)|Bytes|Máximo||
|usedmemoryRss6|Memória RSS usada (Fragmento 6)|Bytes|Máximo||
|serverLoad6|Carga do servidor (Fragmento 6)|Porcentagem|Máximo||
|cacheWrite6|Gravação no cache (Fragmento 6)|BytesPerSecond|Máximo||
|cacheRead6|Leitura no cache (Fragmento 6)|BytesPerSecond|Máximo||
|percentProcessorTime6|CPU (Fragmento 6)|Porcentagem|Máximo||
|connectedclients7|Clientes conectados (Fragmento 7)|Contagem|Máximo||
|totalcommandsprocessed7|Total de operações (Fragmento 7)|Contagem|Total||
|cachehits7|Ocorrências no cache (Fragmento 7)|Contagem|Total||
|cachemisses7|Perdas no cache (Fragmento 7)|Contagem|Total||
|getcommands7|Gets (Fragmento 7)|Contagem|Total||
|setcommands7|Sets (Fragmento 7)|Contagem|Total||
|evictedkeys7|Chaves removidas (Fragmento 7)|Contagem|Total||
|totalkeys7|Total de chaves (Nó 7)|Contagem|Máximo||
|expiredkeys7|Chaves expiradas (Fragmento 7)|Contagem|Total||
|usedmemory7|Memória usada (Fragmento 7)|Bytes|Máximo||
|usedmemoryRss7|Memória RSS usada (Fragmento 7)|Bytes|Máximo||
|serverLoad7|Carga do servidor (Fragmento 7)|Porcentagem|Máximo||
|cacheWrite7|Gravação no cache (Fragmento 7)|BytesPerSecond|Máximo||
|cacheRead7|Leitura no cache (Fragmento 7)|BytesPerSecond|Máximo||
|percentProcessorTime7|CPU (Fragmento 7)|Porcentagem|Máximo||
|connectedclients8|Clientes conectados (Fragmento 8)|Contagem|Máximo||
|totalcommandsprocessed8|Total de operações (Fragmento 8)|Contagem|Total||
|cachehits8|Ocorrências no cache (Fragmento 8)|Contagem|Total||
|cachemisses8|Perdas no cache (Fragmento 8)|Contagem|Total||
|getcommands8|Gets (Fragmento 8)|Contagem|Total||
|setcommands8|Sets (Fragmento 8)|Contagem|Total||
|evictedkeys8|Chaves removidas (Fragmento 8)|Contagem|Total||
|totalkeys8|Total de chaves (Nó 8)|Contagem|Máximo||
|expiredkeys8|Chaves expiradas (Fragmento 8)|Contagem|Total||
|usedmemory8|Memória usada (Fragmento 8)|Bytes|Máximo||
|usedmemoryRss8|Memória RSS usada (Fragmento 8)|Bytes|Máximo||
|serverLoad8|Carga do servidor (Fragmento 8)|Porcentagem|Máximo||
|cacheWrite8|Gravação no cache (Fragmento 8)|BytesPerSecond|Máximo||
|cacheRead8|Leitura no cache (Fragmento 8)|BytesPerSecond|Máximo||
|percentProcessorTime8|CPU (Fragmento 8)|Porcentagem|Máximo||
|connectedclients9|Clientes conectados (Fragmento 9)|Contagem|Máximo||
|totalcommandsprocessed9|Total de operações (Fragmento 9)|Contagem|Total||
|cachehits9|Ocorrências no cache (Fragmento 9)|Contagem|Total||
|cachemisses9|Perdas no cache (Fragmento 9)|Contagem|Total||
|getcommands9|Gets (Fragmento 9)|Contagem|Total||
|setcommands9|Sets (Fragmento 9)|Contagem|Total||
|evictedkeys9|Chaves removidas (Fragmento 9)|Contagem|Total||
|totalkeys9|Total de chaves (Nó 9)|Contagem|Máximo||
|expiredkeys9|Chaves expiradas (Fragmento 9)|Contagem|Total||
|usedmemory9|Memória usada (Fragmento 9)|Bytes|Máximo||
|usedmemoryRss9|Memória RSS usada (Fragmento 9)|Bytes|Máximo||
|serverLoad9|Carga do servidor (Fragmento 9)|Porcentagem|Máximo||
|cacheWrite9|Gravação no cache (Fragmento 9)|BytesPerSecond|Máximo||
|cacheRead9|Leitura no cache (Fragmento 9)|BytesPerSecond|Máximo||
|percentProcessorTime9|CPU (Fragmento 9)|Porcentagem|Máximo||

## Microsoft.Compute/virtualMachines

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|Porcentagem de CPU|Porcentagem de CPU|Porcentagem|Média|A porcentagem das unidades de computação alocadas que estão atualmente em uso pela máquina virtual ou pelas máquinas virtuais|
|Entrada na rede|Entrada na rede|Bytes|Total|O número de bytes recebidos em todas as interfaces de rede pela máquina virtual ou pelas máquinas virtuais (tráfego de entrada)|
|Saída da rede|Saída da rede|Bytes|Total|O número de bytes de saída em todas as interfaces de rede pela máquina virtual ou pelas máquinas virtuais (tráfego de saída)|
|Bytes de leitura de disco|Bytes de leitura de disco|Bytes|Total|Total de bytes lidos do disco durante o período de monitoramento|
|Bytes de gravação de disco|Bytes de gravação de disco|Bytes|Total|Total de bytes gravados em disco durante o período de monitoramento|
|Operações de leitura de disco/Seg|Operações de leitura de disco/Seg|CountPerSecond|Média|IOPS de leitura de disco|
|Operações de gravação de disco/Seg|Operações de gravação de disco/Seg|CountPerSecond|Média|IOPS de gravação de disco|

## Microsoft.Compute/virtualMachineScaleSets

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|Porcentagem de CPU|Porcentagem de CPU|Porcentagem|Média|A porcentagem das unidades de computação alocadas que estão atualmente em uso pela máquina virtual ou pelas máquinas virtuais|
|Entrada na rede|Entrada na rede|Bytes|Total|O número de bytes recebidos em todas as interfaces de rede pela máquina virtual ou pelas máquinas virtuais (tráfego de entrada)|
|Saída da rede|Saída da rede|Bytes|Total|O número de bytes de saída em todas as interfaces de rede pela máquina virtual ou pelas máquinas virtuais (tráfego de saída)|
|Bytes de leitura de disco|Bytes de leitura de disco|Bytes|Total|Total de bytes lidos do disco durante o período de monitoramento|
|Bytes de gravação de disco|Bytes de gravação de disco|Bytes|Total|Total de bytes gravados em disco durante o período de monitoramento|
|Operações de leitura de disco/Seg|Operações de leitura de disco/Seg|CountPerSecond|Média|IOPS de leitura de disco|
|Operações de gravação de disco/Seg|Operações de gravação de disco/Seg|CountPerSecond|Média|IOPS de gravação de disco|

## Microsoft.Compute/virtualMachineScaleSets/virtualMachines

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|Porcentagem de CPU|Porcentagem de CPU|Porcentagem|Média|A porcentagem das unidades de computação alocadas que estão atualmente em uso pela máquina virtual ou pelas máquinas virtuais|
|Entrada na rede|Entrada na rede|Bytes|Total|O número de bytes recebidos em todas as interfaces de rede pela máquina virtual ou pelas máquinas virtuais (tráfego de entrada)|
|Saída da rede|Saída da rede|Bytes|Total|O número de bytes de saída em todas as interfaces de rede pela máquina virtual ou pelas máquinas virtuais (tráfego de saída)|
|Bytes de leitura de disco|Bytes de leitura de disco|Bytes|Total|Total de bytes lidos do disco durante o período de monitoramento|
|Bytes de gravação de disco|Bytes de gravação de disco|Bytes|Total|Total de bytes gravados em disco durante o período de monitoramento|
|Operações de leitura de disco/Seg|Operações de leitura de disco/Seg|CountPerSecond|Média|IOPS de leitura de disco|
|Operações de gravação de disco/Seg|Operações de gravação de disco/Seg|CountPerSecond|Média|IOPS de gravação de disco|

## Microsoft.Devices/IotHubs

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|d2c.telemetry.ingress.allProtocol|Tentativas de envio de mensagem de telemetria|Contagem|Total|Número do dispositivo para tentativas de envio de mensagens de telemetria da nuvem para o seu hub IoT|
|d2c.telemetry.ingress.success|Mensagem de telemetria enviadas|Contagem|Total|Número do dispositivo para mensagens de telemetria da nuvem enviadas com sucesso para o seu hub IoT|
|c2d.commands.egress.complete.success|Comandos concluídos|Contagem|Total|Número de comandos da nuvem para o dispositivo concluídos com sucesso pelo dispositivo|
|c2d.commands.egress.abandon.success|Comandos abandonados|Contagem|Total|Número de comandos da nuvem para o dispositivo abandonados pelo dispositivo|
|c2d.commands.egress.reject.success|Comandos rejeitados|Contagem|Total|Número de comandos da nuvem para o dispositivo rejeitados pelo dispositivo|
|devices.totalDevices|Total de dispositivos|Contagem|Total|Número de dispositivos registrados para o seu hub IoT|
|devices.connectedDevices.allProtocol|Dispositivos conectados|Contagem|Total|Número de dispositivos registrados ao seu hub IoT|

## Microsoft.EventHub/namespaces

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|INREQS|Solicitações de entrada|Contagem|Total||
|SUCCREQ|Solicitações bem sucedidas|Contagem|Total||
|FAILREQ|Solicitações com falha|Contagem|Total||
|SVRBSY|Erros de servidor ocupado|Contagem|Total||
|INTERR|Erros internos do servidor|Contagem|Total||
|MISCERR|Outros erros|Contagem|Total||
|INMSGS|Mensagens de entrada|Contagem|Total||
|OUTMSGS|Mensagens de saída|Contagem|Total||
|EHINMBS|Bytes de entrada por segundo|BytesPerSecond|Total||
|EHOUTMBS|Bytes de saída por segundo|BytesPerSecond|Total||
|EHABL|Arquivar mensagens da lista de pendências|Contagem|Total||
|EHAMSGS|Arquivar mensagens|Contagem|Total||
|EHAMBS|Arquivar taxa de transferência de mensagem|BytesPerSecond|Total||

## Microsoft.Logic/workflows

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|RunsStarted|Execuções iniciadas|Contagem|Total|Número de execuções fluxo de trabalho iniciadas.|
|RunsCompleted|Execuções concluídas|Contagem|Total|Número de execuções de fluxo de trabalho concluídas.|
|RunsSucceeded|Execuções bem sucedidas|Contagem|Total|Número de execuções de fluxo de trabalho bem sucedidas.|
|RunsFailed|Execuções com falha|Contagem|Total|Número de execuções de fluxo de trabalho com falha.|
|RunsCancelled|Execuções canceladas|Contagem|Total|Número de execuções de fluxo de trabalho canceladas.|
|RunLatency|Latência da execução|Segundos|Média|Latência das execuções de fluxo de trabalho concluídas.|
|RunSuccessLatency|Latência de execução bem sucedida|Segundos|Média|Latência de execuções de fluxo de trabalho bem sucedidas.|
|RunThrottledEvents|Eventos de restrição de execução|Contagem|Total|Número de eventos de restrição de ações ou gatilhos de fluxo de trabalho.|
|ActionsStarted|Ações iniciadas |Contagem|Total|Número de ações de fluxo de trabalho iniciadas.|
|ActionsCompleted|Ações concluídas |Contagem|Total|Número de ações de fluxo de trabalho concluídas.|
|ActionsSucceeded|Ações bem sucedidas |Contagem|Total|Número de ações de fluxo de trabalho bem sucedidas.|
|ActionsFailed|Ações com falha|Contagem|Total|Número de ações de fluxo de trabalho com falha.|
|ActionsSkipped|Ações ignoradas |Contagem|Total|Número de ações de fluxo de trabalho ignoradas.|
|ActionLatency|Latência da ação |Segundos|Média|Latência das ações de fluxo de trabalho concluídas.|
|ActionSuccessLatency|Latência das ações bem sucedidas |Segundos|Média|Latência das ações de fluxo de trabalho bem sucedidas.|
|ActionThrottledEvents|Eventos de restrição de ações|Contagem|Total|Número de eventos de restrição de ações de fluxo de trabalho.|
|TriggersStarted|Gatilhos iniciados |Contagem|Total|Número de gatilhos de fluxo de trabalho iniciadas.|
|TriggersCompleted|Gatilhos concluídos |Contagem|Total|Número de gatilhos de fluxo de trabalho concluídos.|
|TriggersSucceeded|Gatilhos bem sucedidos |Contagem|Total|Número de gatilhos de fluxo de trabalho bem sucedidos.|
|TriggersFailed|Gatilhos com falha |Contagem|Total|Número de gatilhos de fluxo de trabalho com falha.|
|TriggersSkipped|Gatilhos ignorados|Contagem|Total|Número de gatilhos de fluxo de trabalho ignorados.|
|TriggersFired|Gatilhos acionados |Contagem|Total|Número de gatilhos de fluxo de trabalho acionados.|
|TriggerLatency|Latência do gatilho |Segundos|Média|Latência dos gatilhos de fluxo de trabalho concluídos.|
|TriggerFireLatency|Latência do gatilho acionado |Segundos|Média|Latência dos gatilhos de fluxo de trabalho acionados.|
|TriggerSuccessLatency|Latência do gatilho bem sucedido |Segundos|Média|Latência dos gatilhos de fluxo de trabalho bem sucedidos.|
|TriggerThrottledEvents|Eventos de restrição do gatilho|Contagem|Total|Número de eventos de restrição do gatilho de fluxo de trabalho.|

## Microsoft.Network/applicationGateways

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|Taxa de transferência|Taxa de transferência|BytesPerSecond|Média||

## Microsoft.Search/searchServices

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|Latência|Latência|Segundos|Média|Latência média do serviço de pesquisa|
|SearchLatency|Latência de pesquisa|Segundos|Média|Latência média de pesquisa do serviço de pesquisa|
|SearchQueriesPerSecond|Consultas de pesquisa por segundo|CountPerSecond|Média|Consultas de pesquisa por segundo para o serviço de pesquisa|
|ThrottledSearchQueriesPercentage|Porcentagem das consultas de pesquisa limitadas|Porcentagem|Média|Porcentagem de consultas de pesquisa que eram limitadas para o serviço de pesquisa|

## Microsoft.ServiceBus/namespaces

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|CPUXNS|Uso da CPU por namespace|Porcentagem|Máximo||
|WSXNS|Uso do tamanho da memória por namespace|Porcentagem|Máximo||

## Microsoft.Sql/servers/databases

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|cpu\_percent|Percentual de CPU|Porcentagem|Máximo|Percentual de CPU|
|physical\_data\_read\_percent|Porcentagem de E/S de dados|Porcentagem|Máximo|Porcentagem de E/S de dados|
|log\_write\_percent|Porcentagem de E/S de log|Porcentagem|Máximo|Porcentagem de E/S de log|
|dtu\_consumption\_percent|Porcentagem de DTU|Porcentagem|Máximo|Porcentagem de DTU|
|storage|Tamanho total do banco de dados|Bytes|Máximo|Tamanho total do banco de dados|
|connection\_successful|Conexões bem sucedidas|Contagem|Total|Conexões bem sucedidas|
|connection\_failed|Conexões com falha|Contagem|Total|Conexões com falha|
|blocked\_by\_firewall|Bloqueado pelo firewall|Contagem|Total|Bloqueado pelo firewall|
|deadlock|Deadlocks|Contagem|Total|Deadlocks|
|storage\_percent|Percentual de tamanho do banco de dados|Porcentagem|Máximo|Percentual de tamanho do banco de dados|
|xtp\_storage\_percent|Percentual (Visualização) de armazenamento do OLTP na memória|Porcentagem|Máximo|Percentual (Visualização) de armazenamento do OLTP na memória|
|workers\_percent|Porcentagem de funcionários|Porcentagem|Máximo|Porcentagem de funcionários|
|sessions\_percent|Porcentagem de sessões|Porcentagem|Máximo|Porcentagem de sessões|
|dtu\_limit|Limite de DTU|Contagem|Máximo|Limite de DTU|
|dtu\_used|DTU usado|Contagem|Máximo|DTU usado|
|service\_level\_objective|Objetivo de nível de serviço do banco de dados|Contagem|Total|Objetivo de nível de serviço do banco de dados|
|dwu\_limit|limite de dwu|Contagem|Máximo|limite de dwu|
|dwu\_consumption\_percent|Porcentagem de DWU|Porcentagem|Máximo|Porcentagem de DWU|
|dwu\_used|DWU usado|Contagem|Máximo|DWU usado|

## Microsoft.Sql/servers/elasticPools

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|cpu\_percent|Percentual de CPU|Porcentagem|Máximo|Percentual de CPU|
|physical\_data\_read\_percent|Porcentagem de E/S de dados|Porcentagem|Máximo|Porcentagem de E/S de dados|
|log\_write\_percent|Porcentagem de E/S de log|Porcentagem|Máximo|Porcentagem de E/S de log|
|dtu\_consumption\_percent|Porcentagem de DTU|Porcentagem|Máximo|Porcentagem de DTU|
|storage\_percent|Porcentagem de armazenamento|Porcentagem|Máximo|Porcentagem de armazenamento|
|workers\_percent|Porcentagem de funcionários|Porcentagem|Máximo|Porcentagem de funcionários|
|sessions\_percent|Porcentagem de sessões|Porcentagem|Máximo|Porcentagem de sessões|
|eDTU\_limit|Limite de eDTU|Contagem|Máximo|Limite de eDTU|
|storage\_limit|Limite de armazenamento|Bytes|Máximo|Limite de armazenamento|
|eDTU\_used|eDTU usado|Contagem|Máximo|eDTU usado|
|storage\_used|Armazenamento usado|Bytes|Máximo|Armazenamento usado|

## Microsoft.StreamAnalytics/streamingjobs

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|ResourceUtilization|% de utilização do SU|Porcentagem|Máximo|% de utilização do SU|
|InputEvents|Eventos de entrada|Bytes|Total|Eventos de entrada|
|InputEventBytes|Bytes de evento de entrada|Bytes|Total|Bytes de evento de entrada|
|LateInputEvents|Eventos de entrada atrasados|Bytes|Total|Eventos de entrada atrasados|
|OutputEvents|Eventos de saída|Bytes|Total|Eventos de saída|
|ConversionErrors|Erros de conversão de dados|Bytes|Total|Erros de conversão de dados|
|Erros|Erros de tempo de execução|Bytes|Total|Erros de tempo de execução|
|DroppedOrAdjustedEvents|Eventos fora de ordem|Bytes|Total|Eventos fora de ordem|
|AMLCalloutRequests|Solicitações de função|Bytes|Total|Solicitações de função|
|AMLCalloutFailedRequests|Solicitações de função com falha|Bytes|Total|Solicitações de função com falha|
|AMLCalloutInputEvents|Eventos de função|Bytes|Total|Eventos de função|

## Microsoft.Web/serverfarms

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|CpuPercentage|Porcentagem de CPU|Porcentagem|Média|Porcentagem de CPU|
|MemoryPercentage|Porcentagem de Memória|Porcentagem|Média|Porcentagem de Memória|
|DiskQueueLength|Tamanho da fila do disco|Contagem|Total|Tamanho da fila do disco|
|HttpQueueLength|Tamanho da Fila de Http|Contagem|Total|Tamanho da Fila de Http|
|BytesReceived|Entrada de Dados|Bytes|Total|Entrada de Dados|
|BytesSent|Saída de dados|Bytes|Total|Saída de dados|

## Microsoft.Web/sites

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|CpuTime|Tempo de CPU|Segundos|Total|Tempo de CPU|
|Solicitações|Solicitações|Contagem|Total|Solicitações|
|BytesReceived|Entrada de Dados|Bytes|Total|Entrada de Dados|
|BytesSent|Saída de dados|Bytes|Total|Saída de dados|
|Http2xx|Http 2xx|Contagem|Total|Http 2xx|
|Http3xx|Http 3xx|Contagem|Total|Http 3xx|
|Http401|Http 401|Contagem|Total|Http 401|
|Http403|Http 403|Contagem|Total|Http 403|
|Http404|Http 404|Contagem|Total|Http 404|
|Http406|Http 406|Contagem|Total|Http 406|
|Http4xx|Http 4xx|Contagem|Total|Http 4xx|
|Http5xx|Erros do Servidor Http|Contagem|Total|Erros do Servidor Http|
|MemoryWorkingSet|Conjunto de trabalho de memória|Bytes|Total|Conjunto de trabalho de memória|
|AverageMemoryWorkingSet|Conjunto de trabalho de memória média|Bytes|Média|Conjunto de trabalho de memória média|
|AverageResponseTime|Tempo Médio de Resposta|Segundos|Média|Tempo Médio de Resposta|

## Microsoft.Web/sites/slots

|Métrica|Nome de exibição da métrica|Unidade|Tipo de agregação|Descrição|
|---|---|---|---|---|
|CpuTime|Tempo de CPU|Segundos|Total|Tempo de CPU|
|Solicitações|Solicitações|Contagem|Total|Solicitações|
|BytesReceived|Entrada de Dados|Bytes|Total|Entrada de Dados|
|BytesSent|Saída de dados|Bytes|Total|Saída de dados|
|Http2xx|Http 2xx|Contagem|Total|Http 2xx|
|Http3xx|Http 3xx|Contagem|Total|Http 3xx|
|Http401|Http 401|Contagem|Total|Http 401|
|Http403|Http 403|Contagem|Total|Http 403|
|Http404|Http 404|Contagem|Total|Http 404|
|Http406|Http 406|Contagem|Total|Http 406|
|Http4xx|Http 4xx|Contagem|Total|Http 4xx|
|Http5xx|Erros do Servidor Http|Contagem|Total|Erros do Servidor Http|
|MemoryWorkingSet|Conjunto de trabalho de memória|Bytes|Total|Conjunto de trabalho de memória|
|AverageMemoryWorkingSet|Conjunto de trabalho de memória média|Bytes|Média|Conjunto de trabalho de memória média|
|AverageResponseTime|Tempo Médio de Resposta|Segundos|Média|Tempo Médio de Resposta|

## Próximas etapas

- [Leia sobre as métricas no Azure Monitor](./monitoring-overview.md#monitoring-sources)
- [Criar alertas para métricas](./insights-receive-alert-notifications.md)
- [Exportar as métricas de armazenamento, Hub de eventos ou Log Analytics](./monitoring-overview-of-diagnostic-logs.md)

<!---HONumber=AcomDC_0928_2016-->
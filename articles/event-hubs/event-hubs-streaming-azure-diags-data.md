<properties
	pageTitle="Transmitindo dados do Diagnóstico do Azure no afunilamento usando os Hubs de Eventos | Microsoft Azure"
	description="Ilustra como configurar o Diagnóstico do Azure com Hubs de Eventos de ponta a ponta, incluindo as diretrizes para cenários comuns."
	services="event-hubs"
	documentationCenter="na"
	authors="sethmanheim"
	manager="timlt"
	editor="" />
<tags
	ms.service="event-hubs"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="na"
	ms.date="07/12/2016"
	ms.author="sethm" />

# Streaming de dados de Diagnóstico do Azure no afunilamento usando os Hubs de Eventos

O Diagnóstico do Azure fornece maneiras flexíveis para coletar as métricas e os logs das VMs (máquinas virtuais) dos serviços de nuvem e para transferir os resultados para o armazenamento do Azure. A partir de março de 2016 (SDK 2.9), você poderá coletar diagnósticos para fontes de dados totalmente personalizadas e transferir dados do afunilamento em questão de segundos usando os Hubs de Eventos do Azure.

Entre os tipos de dados com suporte estão:

- Eventos de ETW (Rastreamento de Eventos para Windows)
- Contadores de desempenho
- Logs de eventos do Windows
- Logs de aplicativo
- Logs de infraestrutura do Diagnóstico do Azure

Este artigo mostrará a configuração completa do Diagnóstico do Azure com os Hubs de Eventos. Também são fornecidas diretrizes para os seguintes cenários comuns:

- Como personalizar as métricas e os logs que são coletados para os Hubs de Eventos
- Como alterar as configurações em cada ambiente
- Como exibir dados de fluxo dos Hubs de Eventos
- Como solucionar problemas de conexão

## Pré-requisitos
Há suporte para a coleta de Hubs de Eventos no Diagnósticos do Azure nos Serviços de Nuvem, nas VMs, nos Conjuntos de escala de Máquina Virtual e no Service Fabric, começando no Azure SDK 2.9 e nas Ferramentas do Azure correspondentes para o Visual Studio.

- Extensão 1.6 do Diagnóstico do Azure (o [SDK do Azure para .NET 2.9 ou posterior](https://azure.microsoft.com/downloads/) resolve isso por padrão)
- [Visual Studio 2013 ou posterior.](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx)
- As configurações existentes do Diagnóstico do Azure em um aplicativo usando um arquivo *.wadcfgx* e um dos seguintes métodos:
	- Visual Studio: [Configurando o diagnóstico dos Serviços de Nuvem e das Máquinas Virtuais do Azure](../vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines.md)
	- Windows PowerShell: [Habilitar o diagnóstico nos Serviços de Nuvem do Azure usando o PowerShell](../cloud-services/cloud-services-diagnostics-powershell.md)
- Namespace de Hubs de Eventos provisionado de acordo com o artigo [Get started with Event Hubs (Introdução aos Hubs de Eventos)](./event-hubs-csharp-ephcs-getstarted.md)

## Conectar o Diagnóstico do Azure no coletor dos Hubs de Eventos
O Diagnóstico do Azure sempre envia logs e métricas, por padrão, a uma conta de Armazenamento do Azure. Além disso, um aplicativo pode coletar nos Hubs de Eventos, adicionando uma nova seção **Coletores** ao elemento **WadCfg**, na seção **PublicConfig** do arquivo *.wadcfgx*. No Visual Studio, o arquivo *.wadcfgx* é armazenado no seguinte caminho: **Projeto do Serviço de Nuvem** > **Funções** > **(RoleName)** > arquivo **diagnostics.wadcfgx**.

	  <SinksConfig>
	    <Sink name="HotPath">
	      <EventHub Url="https://diags-mycompany-ns.servicebus.windows.net/diageventhub" SharedAccessKeyName="SendRule" />
	    </Sink>
	  </SinksConfig>

Neste exemplo, a URL do hub de eventos é definida no namespace totalmente qualificado do hub de eventos: namespace ServiceBus + "/" + nome do hub de eventos.

A URL do hub de eventos é exibida no [Portal do Azure](http://go.microsoft.com/fwlink/?LinkID=213885), no painel Hubs de Eventos.

O nome do **Coletor** pode ser definido como qualquer cadeia de caracteres válida, desde que o mesmo valor seja usado de maneira consistente no arquivo de configuração.

>[AZURE.NOTE]  Pode haver outros coletores configurados nesta seção, como o *applicationInsights*. O Diagnóstico do Azure permitirá a definição de um ou mais coletores, se cada coletor também for declarado na seção **PrivateConfig**.

O coletor dos Hubs de Eventos também deve ser declarado e definido na seção **PrivateConfig** do arquivo de configuração *.wadcfgx*.

	  <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
	    <StorageAccount name="" key="" endpoint="" />
	    <EventHub Url="https://diags-mycompany-ns.servicebus.windows.net/diageventhub" SharedAccessKeyName="SendRule" SharedAccessKey="9B3SwghJOGEUvXigc6zHPInl2iYxrgsKHZoy4nm9CUI=" />
	  </PrivateConfig>

O **SharedAccessKeyName** deve ser compatível com uma chave de SAS (Assinaturas de Acesso Compartilhado) e com uma política definida no namespace **ServiceBus/EventHub**. Navegue até o painel de Hubs de Eventos no [Portal do Azure](https://manage.windowsazure.com), selecione a guia **Configurar** e configure uma política nomeada (por exemplo, "SendRule") que tenha permissões de *Envio*. A **StorageAccount** também é declarada em **PrivateConfig**. Não é necessário alterar os valores aqui, especialmente se eles estiverem funcionando. Neste exemplo, deixamos os valores vazios, o que é um sinal de que um ativo de downstream definirá os valores. Por exemplo, o arquivo de configuração do ambiente *ServiceConfiguration.Cloud.cscfg* definirá as chaves e os nomes apropriados do ambiente.

>[AZURE.WARNING] Lembre-se de que a chave de SAS do hub de eventos é armazenada em texto sem formatação no arquivo *.wadcfgx*. Muitas vezes, esta chave é verificada no controle do código-fonte ou está disponível como um ativo no servidor de build e, portanto, você deve protegê-la de maneira apropriada. Recomendamos o uso de uma chave de SAS com permissões do tipo *Somente envio*, de forma que um usuário mal-intencionado poderá gravar no hub de eventos, mas nunca poderá ouvir nem gerenciá-lo.

## Configurar logs e métricas do Diagnóstico do Azure para o coletor com os Hubs de Eventos
Como discutido anteriormente, todos os dados de diagnóstico padrão e personalizados, ou seja, as métricas e os logs, são automaticamente coletados no Armazenamento do Azure em intervalos configurados. Com os Hubs de Eventos e com qualquer outro coletor, você pode especificar a coleta de qualquer nó raiz ou folha na hierarquia com o hub de eventos. Isso inclui eventos de ETW, contadores de desempenho, logs de eventos do Windows e logs de aplicativo.

É importante considerar quantos pontos de dados devem ser transferidos para os Hubs de Eventos. Normalmente, os desenvolvedores transferem dados de caminhos recorrentes e de baixa latência, que devem ser consumidos e interpretados rapidamente. Os sistemas que monitoram os alertas ou as regras de dimensionamento automático são exemplos. Um desenvolvedor também pode configurar um repositório de análise alternativo ou um repositório de pesquisa – por exemplo, o Stream Analytics do Azure, o Elasticsearch, um sistema de monitoramento personalizado ou um sistema de monitoramento favorito de outros usuários.

Veja a seguir alguns exemplos de configurações.

        <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="HotPath">
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\Bytes Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Requests/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Errors Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Queued" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Rejected" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" />
        </PerformanceCounters>

No exemplo a seguir, o coletor é aplicado ao nó pai **PerformanceCounters** na hierarquia, o que significa que todos os **PerformanceCounters** filho serão enviados aos Hubs de Eventos.

        <PerformanceCounters scheduledTransferPeriod="PT1M">
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\Bytes Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Requests/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Errors Total/Sec" sampleRate="PT3M" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Queued" sampleRate="PT3M" sinks="HotPath" />
          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Rejected" sampleRate="PT3M" sinks="HotPath"/>
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" sinks="HotPath"/>
        </PerformanceCounters>

No exemplo anterior, o coletor é aplicado a apenas três contadores: Solicitações Enfileiradas, Solicitações Rejeitadas e % do Tempo do processador.

O exemplo a seguir mostra como um desenvolvedor pode limitar a quantidade de dados enviados, a fim de definir as métricas essenciais que são usadas para a integridade deste serviço.

        <Logs scheduledTransferPeriod="PT1M" sinks="HotPath" scheduledTransferLogLevelFilter="Error" />

Neste exemplo, o coletor é aplicado aos logs e é filtrado apenas para o rastreamento de nível de erro.

## Implantar e atualizar uma configuração de aplicativo e diagnóstico dos Serviços de Nuvem

O Visual Studio fornece o caminho mais fácil para implantar o aplicativo e a configuração do coletor de Hubs de Eventos. Para exibir e editar o arquivo, abra o arquivo *.wadcfgx* no Visual Studio, edite e salve-o. O caminho é **Projeto de Serviço de Nuvem** > **Funções** > **(RoleName)** > **diagnostics.wadcfgx**.

Neste ponto, todas as implantações e ações de atualização de implantações no Visual Studio, no Visual Studio Team System e em todos os comandos ou scripts que são baseados no MSBuild e usam o destino /t:publish incluirão o *.wadcfgx* no processo de empacotamento. Além disso, as implantações e atualizações implantarão o arquivo no Azure usando a extensão apropriada do agente de diagnóstico do Azure em suas VMs.

Depois de implantar o aplicativo e a configuração do Diagnóstico do Azure, você imediatamente verá a atividade no painel do hub de eventos. Isso indica que você está pronto para começar a ver os dados do afunilamento no cliente ouvinte ou na ferramenta de análise de sua escolha.

Na figura a seguir, o painel do hub de eventos mostra os envios íntegros dos dados de diagnóstico para o hub de eventos começando em algum momento depois das 23:00 h. Ou seja, quando o aplicativo foi implantado com um arquivo *.wadcfgx* atualizado e o coletor foi configurado corretamente.

![][0]

>[AZURE.NOTE] Quando você faz atualizações no arquivo de configuração do Diagnóstico do Azure (.wadcfgx), recomendamos o envio por push das atualizações para todo o aplicativo e também a configuração usando a publicação do Visual Studio ou o script do Windows PowerShell.

## Exibir dados de caminhos recorrentes

Como discutido anteriormente, há muitos casos de uso sobre ouvir e processar os dados dos Hubs de Eventos.

Uma abordagem simples é criar um pequeno aplicativo de console de teste para ouvir o hub de eventos e imprimir a transmissão de saída. Você pode colocar o código a seguir, que é explicado em mais detalhes na [Introdução aos Hubs de Eventos](./event-hubs-csharp-ephcs-getstarted.md), em um aplicativo de console.

Observe que o aplicativo de console deve incluir o [pacote EventProcessor do Nuget](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/).

Lembre-se de substituir os valores entre colchetes na função **Principal** pelos valores de seus recursos.

	//Console application code for EventHub test client
	using System;
	using System.Collections.Generic;
	using System.Diagnostics;
	using System.Linq;
	using System.Text;
	using System.Threading.Tasks;
	using Microsoft.ServiceBus.Messaging;

	namespace EventHubListener
	{
	    class SimpleEventProcessor : IEventProcessor
	    {
	        Stopwatch checkpointStopWatch;

	        async Task IEventProcessor.CloseAsync(PartitionContext context, CloseReason reason)
	        {
	            Console.WriteLine("Processor Shutting Down. Partition '{0}', Reason: '{1}'.", context.Lease.PartitionId, reason);
	            if (reason == CloseReason.Shutdown)
	            {
	                await context.CheckpointAsync();
	            }
	        }

	        Task IEventProcessor.OpenAsync(PartitionContext context)
	        {
	            Console.WriteLine("SimpleEventProcessor initialized.  Partition: '{0}', Offset: '{1}'", context.Lease.PartitionId, context.Lease.Offset);
	            this.checkpointStopWatch = new Stopwatch();
	            this.checkpointStopWatch.Start();
	            return Task.FromResult<object>(null);
	        }

	        async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
	        {
	            foreach (EventData eventData in messages)
	            {
	                string data = Encoding.UTF8.GetString(eventData.GetBytes());

	                Console.WriteLine(string.Format("Message received.  Partition: '{0}', Data: '{1}'",
	                    context.Lease.PartitionId, data));

	                foreach (var x in eventData.Properties)
	                {
	                    Console.WriteLine(string.Format("    {0} = {1}", x.Key, x.Value));
	                }
	            }

	            //Call checkpoint every 5 minutes, so that worker can resume processing from 5 minutes back if it restarts.
	            if (this.checkpointStopWatch.Elapsed > TimeSpan.FromMinutes(5))
	            {
	                await context.CheckpointAsync();
	                this.checkpointStopWatch.Restart();
	            }
	        }
	    }

	    class Program
	    {
	        static void Main(string[] args)
	        {
	            string eventHubConnectionString = "Endpoint= <Your Connection String>”
	            string eventHubName = "<EventHub Name>";
	            string storageAccountName = "<Storage Account Name>";
	            string storageAccountKey = "<Storage Account Key>”;
	            string storageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", storageAccountName, storageAccountKey);

	            string eventProcessorHostName = Guid.NewGuid().ToString();
	            EventProcessorHost eventProcessorHost = new EventProcessorHost(eventProcessorHostName, eventHubName, EventHubConsumerGroup.DefaultGroupName, eventHubConnectionString, storageConnectionString);
	            Console.WriteLine("Registering EventProcessor...");
	            var options = new EventProcessorOptions();
	            options.ExceptionReceived += (sender, e) => { Console.WriteLine(e.Exception); };
	            eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>(options).Wait();

	            Console.WriteLine("Receiving. Press enter key to stop worker.");
	            Console.ReadLine();
	            eventProcessorHost.UnregisterEventProcessorAsync().Wait();
	        }
	    }
	}

## Solucionar problemas do coletor dos Hubs de Eventos

- O hub de eventos não mostra a atividade de entrada ou saída de eventos conforme o esperado.

	Verifique se o seu hub de eventos foi provisionado com êxito. Todas as informações de conexão na seção **PrivateConfig** de *.wadcfgx* devem corresponder aos valores de seu recurso, como visto no portal. Verifique se você tem uma política SAS definida ("SendRule", no exemplo) no portal e se a permissão de *Envio* foi concedida.

- Depois de uma atualização, o hub de eventos não mostrará mais a atividade de entrada ou saída dos eventos.

	Primeiro, verifique se as informações do hub de eventos e de configuração estão corretas, conforme explicado anteriormente. Às vezes, a **PrivateConfig** é redefinida em uma atualização de implantação. A solução recomendada é fazer todas as alterações em *.wadcfgx* no projeto e, depois, enviar uma atualização completa do aplicativo por push. Se isso não for possível, verifique se a atualização do diagnóstico enviará por push uma **PrivateConfig** completa, que inclui a chave SAS.

- Eu testei as sugestões, mas o hub de eventos ainda não está funcionando.

	Tente examinar a tabela do Armazenamento do Azure que contém logs e erros do próprio diagnóstico do Azure: **WADDiagnosticInfrastructureLogsTable**. Uma opção é usar uma ferramenta, como o [Gerenciador de Armazenamento do Azure](http://www.storageexplorer.com), para se conectar a esta conta de armazenamento, exibir esta tabela e adicionar uma consulta por carimbo de data/hora nas últimas 24 horas. Você pode usar a ferramenta para exportar um arquivo .csv e abri-lo em um aplicativo, como o Microsoft Excel. O Excel facilita a pesquisa de cadeias de caracteres de cartão, como EventHubs, para ver qual erro é relatado.

## Próximas etapas
• [Saiba mais sobre os Hubs de Eventos](https://azure.microsoft.com/services/event-hubs/)

## Apêndice: Concluir o exemplo de arquivo de configuração do Diagnóstico do Azure (.wadcfgx)
	<?xml version="1.0" encoding="utf-8"?>
	<DiagnosticsConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
	  <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
	    <WadCfg>
	      <DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="applicationInsights.errors">
	        <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error" />
	        <Directories scheduledTransferPeriod="PT1M">
	          <IISLogs containerName="wad-iis-logfiles" />
	          <FailedRequestLogs containerName="wad-failedrequestlogs" />
	        </Directories>
	        <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="HotPath">
	          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\ISAPI Extension Requests/sec" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\Web Service(_Total)\Bytes Total/Sec" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Requests/Sec" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET Applications(__Total__)\Errors Total/Sec" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Queued" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\ASP.NET\Requests Rejected" sampleRate="PT3M" />
	          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT3M" />
	        </PerformanceCounters>
	        <WindowsEventLog scheduledTransferPeriod="PT1M">
	          <DataSource name="Application!*" />
	        </WindowsEventLog>
	        <CrashDumps>
	          <CrashDumpConfiguration processName="WaIISHost.exe" />
	          <CrashDumpConfiguration processName="WaWorkerHost.exe" />
	          <CrashDumpConfiguration processName="w3wp.exe" />
	        </CrashDumps>
	        <Logs scheduledTransferPeriod="PT1M" sinks="HotPath" scheduledTransferLogLevelFilter="Error" />
	      </DiagnosticMonitorConfiguration>
	      <SinksConfig>
	        <Sink name="HotPath">
	          <EventHub Url="https://diageventhub-py-ns.servicebus.windows.net/diageventhub-py" SharedAccessKeyName="SendRule" />
	        </Sink>
	        <Sink name="applicationInsights">
	          <ApplicationInsights />
	          <Channels>
	            <Channel logLevel="Error" name="errors" />
	          </Channels>
	        </Sink>
	      </SinksConfig>
	    </WadCfg>
	    <StorageAccount />
	  </PublicConfig>
	  <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
	    <StorageAccount name="" key="" endpoint="" />
	    <EventHub Url="https://diageventhub-py-ns.servicebus.windows.net/diageventhub-py" SharedAccessKeyName="SendRule" SharedAccessKey="YOUR_KEY_HERE" />
	  </PrivateConfig>
	  <IsEnabled>true</IsEnabled>
	</DiagnosticsConfiguration>

O *ServiceConfiguration.Cloud.cscfg* complementar para este exemplo se parece com o seguinte.

	<?xml version="1.0" encoding="utf-8"?>
	<ServiceConfiguration serviceName="MyFixItCloudService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="3" osVersion="*" schemaVersion="2015-04.2.6">
	  <Role name="MyFixIt.WorkerRole">
	    <Instances count="1" />
	    <ConfigurationSettings>
	      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="YOUR_CONNECTION_STRING" />
	    </ConfigurationSettings>
	  </Role>
	</ServiceConfiguration>

<!-- Images. -->
[0]: ./media/event-hubs-streaming-azure-diags-data/dashboard.png

<!---HONumber=AcomDC_0713_2016-->
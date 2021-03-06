<properties
   pageTitle="Testando os Azure Functions | Microsoft Azure"
   description="Teste os Azure Functions usando o Postman, cURL e Node.js."
   services="functions"
   documentationCenter="na"
   authors="wesmc7777"
   manager="erikre"
   editor=""
   tags=""
   keywords="azure functions, funções, processamento de eventos, webhooks, computação dinâmica, arquitetura sem servidor, testes"/>

<tags
   ms.service="functions"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="multiple"
   ms.workload="na"
   ms.date="08/19/2016"
   ms.author="wesmc"/>

# Testando os Azure Functions

## Visão geral

Neste tutorial, trataremos de diferentes abordagens para testar funções. Definiremos uma função de gatilho http que aceita entrada por meio de um parâmetro de cadeia de caracteres de consulta ou o corpo da solicitação. O código do modelo da **função Node.js HttpTrigger** dá suporte a um parâmetro de cadeia de caracteres de consulta `name`. Também podemos adicionar código para dar suporte a esse parâmetro junto com as informações do `address` para o usuário no corpo da solicitação.

## Criar uma função para teste

Durante a maior parte deste tutorial, usaremos uma versão ligeiramente modificada do modelo da **função Nodejs HttpTrigger** disponível ao criar uma nova função. Você poderá examinar o [Tutorial para criar sua primeira Função do Azure](functions-create-first-azure-function.md) se precisar de ajuda para criar uma nova função. Basta escolher o modelo da **Função Nodejs HttpTrigger** ao criar a função de teste no [Portal do Azure].

O modelo da função padrão é basicamente uma função de Olá, mundo que retorna o nome do corpo da solicitação do parâmetro de cadeia de caracteres da consulta, `name=<your name>`. Atualizaremos o código para permitir também que você forneça o nome e um endereço como conteúdo JSON no corpo da solicitação. Em seguida, a função os retornará para o cliente quando disponível.

Atualize a função com o seguinte código que usaremos para teste:

	module.exports = function(context, req) {
	    context.log("Node.js HTTP trigger function processed a request. RequestUri=%s", req.originalUrl);
	    context.log("Request Headers = " + JSON.stringify(req.headers));    
	
	    if (req.query.name || (req.body && req.body.name)) {
	        if (typeof req.query.name != "undefined") {
	            context.log("Name was provided as a query string param..."); 
	            ProcessNewUserInformation(context, req.query.name);
	        }
	        else {
	            context.log("Processing user info from request body..."); 
	            ProcessNewUserInformation(context, req.body.name, req.body.address);
	        }
	    }
	    else {
	        context.res = {
	            status: 400,
	            body: "Please pass a name on the query string or in the request body"
	        };
	    }
	    context.done();
	};
	
	function ProcessNewUserInformation(context, name, address)
	{    
	    context.log("Processing User Information...");            
	    context.log("name = " + name);            
	    echoString = "Hello " + name;
	    
	    if (typeof address != "undefined")
	    {
	        echoString += "\n" + "The address you provided is " + address;
	        context.log("address = " + address);            
	    }
	    
	    context.res = {
	            // status: 200, /* Defaults to 200 */
	            body: echoString
	        };
	}


## Testar uma função com Ferramentas

### Testar com cURL

Muitas vezes ao testar software, não é necessário procurar além da linha de comando para ajudar a depurar seu aplicativo, e isso não é diferente com funções.

Para testar a função acima, copie o **URL da Função** do portal. Ela estará no seguinte formato:

    https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>
    
Esta é a URL para disparar sua função; podemos testar isso usando o comando cURL na linha de comando para fazer uma solicitação Get (`-G` ou `--get`) em nossa função:

    curl -G https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>
    
Esse exemplo específico acima exige um parâmetro de cadeia de caracteres de consulta que pode ser passado como Dados (`-d`) no comando cURL:

    curl -G https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code> -d name=<Enter a name here>
    
Pressione Enter e você verá a saída da função na linha de comando.

![](./media/functions-test-a-function/curl-test.png)

Na janela **Logs** do portal, é registrada uma saída semelhante à seguinte ao executar a função:

    2016-04-05T21:55:09  Welcome, you are now connected to log-streaming service.
    2016-04-05T21:55:30.738 Function started (Id=ae6955da-29db-401a-b706-482fcd1b8f7a)
    2016-04-05T21:55:30.738 Node.js HTTP trigger function processed a request. RequestUri=https://functionsExample.azurewebsites.net/api/HttpTriggerNodeJS1?code=XXXXXXX&name=Azure Functions
    2016-04-05T21:55:30.738 Function completed (Success, Id=ae6955da-29db-401a-b706-482fcd1b8f7a)

### Testar com um navegador

Funções que não requerem parâmetros, ou precisam apenas de parâmetros de cadeia de caracteres de consulta, podem ser testadas usando um navegador.

Para testar a função que definimos acima, copie a **URL da Função** do portal. Ela estará no seguinte formato:

	https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>

Acrescente o parâmetro de cadeia de caracteres de consulta `name` da seguinte maneira, usando um nome real para o espaço reservado `<Enter a name here>`.

	https://<Your Function App>.azurewebsites.net/api/<Your Function Name>?code=<your access code>&name=<Enter a name here>

Cole a URL no seu navegador e você obterá uma resposta semelhante à seguinte.

![](./media/functions-test-a-function/browser-test.png)

Na janela **Logs** do portal, é registrada uma saída semelhante à seguinte ao executar a função:

	2016-03-23T07:34:59  Welcome, you are now connected to log-streaming service.
	2016-03-23T07:35:09.195 Function started (Id=61a8c5a9-5e44-4da0-909d-91d293f20445)
	2016-03-23T07:35:10.338 Node.js HTTP trigger function processed a request. RequestUri=https://functionsExample.azurewebsites.net/api/WesmcHttpTriggerNodeJS1?code=XXXXXXXXXX==&name=Wes from a browser
	2016-03-23T07:35:10.338 Request Headers = {"cache-control":"max-age=0","connection":"Keep-Alive","accept":"text/html","accept-encoding":"gzip","accept-language":"pt-BR"}
	2016-03-23T07:35:10.338 Name was provided as a query string param.
	2016-03-23T07:35:10.338 Processing User Information...
	2016-03-23T07:35:10.369 Function completed (Success, Id=61a8c5a9-5e44-4da0-909d-91d293f20445)

### Testar com o Postman

A ferramenta recomendada para a testar maioria das funções é o Postman. Para instalar o Postman, consulte [Obter o Postman](https://www.getpostman.com/). O Postman fornece controle sobre muito mais atributos de uma solicitação HTTP.

> [AZURE.TIP] Use o cliente REST com o qual você está familiarizado. Aqui estão algumas alternativas ao Postman:
> 
> * [Fiddler](http://www.telerik.com/fiddler)
> * [Paw](https://luckymarmot.com/paw)

Para testar a função com um corpo de solicitação no Postman:

1. Inicie o Postman com o botão **Aplicativos** no canto superior esquerdo de uma janela do navegador Chrome.
2. Copie a **URL da Função** e cole-a no Postman. Ele inclui o parâmetro de cadeia de caracteres de consulta do código de acesso.
3. Altere o método HTTP para **POST**.
4. Clique em **Corpo** > **bruto** e adicione o corpo da solicitação JSON da seguinte maneira:

		{
		    "name" : "Wes testing with Postman",
		    "address" : "Seattle, W.A. 98101"
		}

5. Clique em **Enviar**.

A imagem a seguir mostra como testar o exemplo de função de eco simples neste tutorial.

![](./media/functions-test-a-function/postman-test.png)

Na janela **Logs** do portal, é registrada uma saída semelhante à seguinte ao executar a função:

	2016-03-23T08:04:51  Welcome, you are now connected to log-streaming service.
	2016-03-23T08:04:57.107 Function started (Id=dc5db8b1-6f1c-4117-b5c4-f6b602d538f7)
	2016-03-23T08:04:57.763 Node.js HTTP trigger function processed a request. RequestUri=https://functions841def78.azurewebsites.net/api/WesmcHttpTriggerNodeJS1?code=XXXXXXXXXX==
	2016-03-23T08:04:57.763 Request Headers = {"cache-control":"no-cache","connection":"Keep-Alive","accept":"*/*","accept-encoding":"gzip","accept-language":"pt-BR"}
	2016-03-23T08:04:57.763 Processing user info from request body...
	2016-03-23T08:04:57.763 Processing User Information...
	2016-03-23T08:04:57.763 name = Wes testing with Postman
	2016-03-23T08:04:57.763 address = Seattle, W.A. 98101
	2016-03-23T08:04:57.795 Function completed (Success, Id=dc5db8b1-6f1c-4117-b5c4-f6b602d538f7)
    
### Testar um gatilho de blob usando o Gerenciador de Armazenamento

Você pode testar uma função de gatilho de blob usando o [Gerenciador de Armazenamento do Microsoft Azure](http://storageexplorer.com/).

1. No [Portal do Azure] para seu aplicativo Functions, crie uma nova função de gatilho F# ou Blob do nó. Defina o caminho a ser monitorado para o nome do seu contêiner de blobs. Por exemplo:

		files

2. Clique no botão **+** para selecionar ou criar a conta de armazenamento que você deseja usar. Em seguida, clique em **Criar**.

3. Crie um arquivo de texto com o seguinte texto e salve-o:

		A text file for blob trigger function testing.

4. Execute o [Gerenciador de Armazenamento do Microsoft Azure](http://storageexplorer.com/) e conecte-se ao contêiner de blob na conta de armazenamento que está sendo monitorada.

5. Clique no botão **Carregar** e carregue o arquivo de texto.

	![](./media/functions-test-a-function/azure-storage-explorer-test.png)


	O código de função do gatilho de blob padrão relatará o processamento do blob nos logs:

		2016-03-24T11:30:10  Welcome, you are now connected to log-streaming service.
		2016-03-24T11:30:34.472 Function started (Id=739ebc07-ff9e-4ec4-a444-e479cec2e460)
		2016-03-24T11:30:34.472 C# Blob trigger function processed: A text file for blob trigger function testing.
		2016-03-24T11:30:34.472 Function completed (Success, Id=739ebc07-ff9e-4ec4-a444-e479cec2e460)

## Testar uma função dentro de funções

### Testar com o botão Executar do portal de funções

O portal fornece um botão **Executar** que permitirá que você faça alguns testes limitados. Você pode fornecer um corpo de solicitação usando o botão Executar, porém não é possível fornecer parâmetros de cadeia de caracteres de consulta ou atualizar os cabeçalhos de solicitação.

Teste a função de gatilho HTTP criada anteriormente adicionando uma cadeia de caracteres JSON similar ao seguinte no campo **Corpo da solicitação** e clique no botão **Executar**.

	{
		"name" : "Wes testing Run button",
		"address" : "USA"
	} 

Na janela **Logs** do portal, é registrada uma saída semelhante à seguinte ao executar a função:

	2016-03-23T08:03:12  Welcome, you are now connected to log-streaming service.
	2016-03-23T08:03:17.357 Function started (Id=753a01b0-45a8-4125-a030-3ad543a89409)
	2016-03-23T08:03:18.697 Node.js HTTP trigger function processed a request. RequestUri=https://functions841def78.azurewebsites.net/api/wesmchttptriggernodejs1
	2016-03-23T08:03:18.697 Request Headers = {"connection":"Keep-Alive","accept":"*/*","accept-encoding":"gzip","accept-language":"pt-BR"}
	2016-03-23T08:03:18.697 Processing user info from request body...
	2016-03-23T08:03:18.697 Processing User Information...
	2016-03-23T08:03:18.697 name = Wes testing Run button
	2016-03-23T08:03:18.697 address = USA
	2016-03-23T08:03:18.744 Function completed (Success, Id=753a01b0-45a8-4125-a030-3ad543a89409)


### Testar com um gatilho de temporizador

Para algumas funções, realmente não é possível testar com as ferramentas mencionadas anteriormente. Por exemplo, uma função de gatilho de fila que é executada quando uma mensagem é inserida no [Armazenamento de Filas do Azure](../storage/storage-dotnet-how-to-use-queues.md). Você sempre pode escrever código para remover uma mensagem da fila e um exemplo disso em um projeto de console é fornecido abaixo. No entanto, há outra abordagem que você pode usar para testar com funções diretamente.

Você pode usar um gatilho de temporizador configurado com uma associação de saída da fila. Esse código de gatilho de temporizador poderia então escrever as mensagens de teste para a fila. Esta seção acompanhará um exemplo passo a passo.

Para obter informações mais detalhadas sobre como usar associações com os Azure Functions, consulte a [Referência do desenvolvedor dos Azure Functions](functions-reference.md).


#### Criar gatilho de fila para teste

Para demonstrar essa abordagem, primeiro criaremos uma função de gatilho de fila que desejamos testar para uma fila denominada `queue-newusers`. Esta função processará as informações de nome e endereço de um novo usuário descartado no Armazenamento de Filas do Azure.

> [AZURE.NOTE] Se você usar um nome de fila diferente, verifique se o nome usado está de acordo com as regras de [Nomeação de Filas e Metadados](https://msdn.microsoft.com/library/dd179349.aspx). Caso contrário, você receberá um código de status HTTP 400: solicitação incorreta.

1. No [Portal do Azure] para seu aplicativo de Funções, clique em **Nova Função** > **QueueTrigger - C#**.
2. Inserir o nome da fila a ser monitorada pela função de fila

		queue-newusers 

3. Clique no botão **+** (adicionar) para selecionar ou criar a conta de armazenamento que você deseja usar. Em seguida, clique em **Criar**.
4. Deixe essa janela do navegador do portal aberta para que você possa monitorar as entradas de log para o código do modelo da função de fila padrão.


#### Criar um gatilho de temporizador para remover uma mensagem na fila

1. Abra o [Portal do Azure] em uma nova janela do navegador e navegue para seu aplicativo de Função.
2. Clique em **Nova Função** > **TimerTrigger - C#**. Digite uma expressão cron para definir com que frequência o código do temporizador executará testes na sua função de fila. Em seguida, clique em **Criar**. Se você quiser que o teste seja executado cada 30 segundos, use a seguinte [expressão CRON](https://wikipedia.org/wiki/Cron#CRON_expression):

		*/30 * * * * *


2. Clique na guia **Integrar** para o novo gatilho de temporizador.
3. Em **Saída**, clique no botão **+ Nova Saída**. Em seguida, clique na **fila** e no botão **Selecionar**.
4. Observe o nome usado para o **objeto de mensagem da fila**; você o usará no código da função de temporizador.

		myQueue

4. Insira o nome da fila na qual a mensagem será enviada:

		queue-newusers 

3. Clique no botão **+** (adicionar) para selecionar a conta de armazenamento que você usou anteriormente com o gatilho de fila. Em seguida, clique em **Salvar**.
4. Clique na guia **Desenvolver** do gatilho de temporizador.
5. Você pode usar o código a seguir para a função de temporizador C#, desde que tenha suado o mesmo nome de objeto de mensagem de fila mostrado acima. Em seguida, clique em **Salvar**

		using System;
		
		public static void Run(TimerInfo myTimer, out String myQueue, TraceWriter log)
		{
		    String newUser = 
		    "{"name":"User testing from C# timer function","address":"XYZ"}";
		
		    log.Verbose($"C# Timer trigger function executed at: {DateTime.Now}");   
		    log.Verbose($"{newUser}");   
		    
		    myQueue = newUser;
		}

Neste ponto, a função de temporizador C# será executada a cada 30 segundos se você usou a expressão cron de exemplo. Os logs para a função de temporizador relatarão cada execução:

	2016-03-24T10:27:02  Welcome, you are now connected to log-streaming service.
	2016-03-24T10:27:30.004 Function started (Id=04061790-974f-4043-b851-48bd4ac424d1)
	2016-03-24T10:27:30.004 C# Timer trigger function executed at: 3/24/2016 10:27:30 AM
	2016-03-24T10:27:30.004 {"name":"User testing from C# timer function","address":"XYZ"}
	2016-03-24T10:27:30.004 Function completed (Success, Id=04061790-974f-4043-b851-48bd4ac424d1)

Na janela do navegador para a função de fila, você verá a cada mensagem que está sendo processada:

	2016-03-24T10:27:06  Welcome, you are now connected to log-streaming service.
	2016-03-24T10:27:30.607 Function started (Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)
	2016-03-24T10:27:30.607 C# Queue trigger function processed: {"name":"User testing from C# timer function","address":"XYZ"}
	2016-03-24T10:27:30.607 Function completed (Success, Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)
	
## Testar uma função com Código

### Testar uma função de gatilho HTTP com Código: Node.js

Você pode usar o código Node.js para executar uma solicitação http para testar sua Função do Azure.

Lembre-se de definir:

- O `host` nas opções de solicitação para o host do aplicativo de função
- O nome da função no `path`.
- Seu código de acesso (`<your code>`) em `path`.

Exemplo de Código:

	var http = require("http");
	
	var nameQueryString = "name=Wes%20Query%20String%20Test%20From%20Node.js";
	
	var nameBodyJSON = {
	    name : "Wes testing with Node.JS code",
	    address : "Dallas, T.X. 75201"
	};
	
	var bodyString = JSON.stringify(nameBodyJSON);
	
	var options = {
	  host: "functions841def78.azurewebsites.net",
	  //path: "/api/HttpTriggerNodeJS2?code=sc1wt62opn7k9buhrm8jpds4ikxvvj42m5ojdt0p91lz5jnhfr2c74ipoujyq26wab3wk5gkfbt9&" + nameQueryString,
	  path: "/api/HttpTriggerNodeJS2?code=sc1wt62opn7k9buhrm8jpds4ikxvvj42m5ojdt0p91lz5jnhfr2c74ipoujyq26wab3wk5gkfbt9",
	  method: "POST",
	  headers : {
	      "Content-Type":"application/json",
	      "Content-Length": Buffer.byteLength(bodyString)
	    }    
	};
	
	callback = function(response) {
	  var str = ""
	  response.on("data", function (chunk) {
	    str += chunk;
	  });
	
	  response.on("end", function () {
	    console.log(str);
	  });
	}
	
	var req = http.request(options, callback);
	console.log("*** Sending name and address in body ***");
	console.log(bodyString);
	req.end(bodyString);



Saída:

	C:\Users\Wesley\testing\Node.js>node testHttpTriggerExample.js
	*** Sending name and address in body ***
	{"name" : "Wes testing with Node.JS code","address" : "Dallas, T.X. 75201"}
	Hello Wes testing with Node.JS code
	The address you provided is Dallas, T.X. 75201
		
Na janela **Logs** do portal, é registrada uma saída semelhante à seguinte ao executar a função:

	2016-03-23T08:08:55  Welcome, you are now connected to log-streaming service.
	2016-03-23T08:08:59.736 Function started (Id=607b891c-08a1-427f-910c-af64ae4f7f9c)
	2016-03-23T08:09:01.153 Node.js HTTP trigger function processed a request. RequestUri=http://functionsExample.azurewebsites.net/api/WesmcHttpTriggerNodeJS1/?code=XXXXXXXXXX==
	2016-03-23T08:09:01.153 Request Headers = {"connection":"Keep-Alive","host":"functionsExample.azurewebsites.net"}
	2016-03-23T08:09:01.153 Name not provided as query string param. Checking body...
	2016-03-23T08:09:01.153 Request Body Type = object
	2016-03-23T08:09:01.153 Request Body = [object Object]
	2016-03-23T08:09:01.153 Processing User Information...
	2016-03-23T08:09:01.215 Function completed (Success, Id=607b891c-08a1-427f-910c-af64ae4f7f9c)
    

### Testar uma função de gatilho de fila com Código: C# #

Mencionamos anteriormente que você poderia testar um gatilho de fila usando código para remover uma mensagem na fila. O exemplo de código a seguir é baseado no código C# apresentado no tutorial [Introdução ao Armazenamento de Filas do Azure](../storage/storage-dotnet-how-to-use-queues.md). Também há código para outras linguagens disponível nesse link.

Para testar esse código em um aplicativo de console, que você deve:

- [Configurar a cadeia de conexão de armazenamento no arquivo app.config](../storage/storage-dotnet-how-to-use-queues.md#setup-a-storage-connection-string).
- Este código aceita o nome e o endereço de um novo usuário como argumentos de linha de comando durante o tempo de execução. Passe um `name` e `address` como parâmetros para o aplicativo. Por exemplo, `C:\myQueueConsoleApp\test.exe "Wes testing queues" "in a console app"`


Código C# de exemplo:

    static void Main(string[] args)
    {
        string name = null;
        string address = null;
        string queueName = "queue-newusers";
        string JSON = null;

        if (args.Length > 0)
        {
            name = args[0];
        }
        if (args.Length > 1)
        {
            address = args[1];
        }

        // Retrieve storage account from connection string
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConfigurationManager.AppSettings["StorageConnectionString"]);

        // Create the queue client
        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

        // Retrieve a reference to a queue
        CloudQueue queue = queueClient.GetQueueReference(queueName);

        // Create the queue if it doesn't already exist
        queue.CreateIfNotExists();

        // Create a message and add it to the queue.
        if (name != null)
        {
            if (address != null)
                JSON = String.Format("{{"name":"{0}","address":"{1}"}}", name, address);
            else
                JSON = String.Format("{{"name":"{0}"}}", name);
        }

        Console.WriteLine("Adding message to " + queueName + "...");
        Console.WriteLine(JSON);

        CloudQueueMessage message = new CloudQueueMessage(JSON);
        queue.AddMessage(message);
    }

Na janela do navegador para a função de fila, você verá a cada mensagem que está sendo processada:

	2016-03-24T10:27:06  Welcome, you are now connected to log-streaming service.
	2016-03-24T10:27:30.607 Function started (Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)
	2016-03-24T10:27:30.607 C# Queue trigger function processed: {"name":"Wes testing queues","address":"in a console app"}
	2016-03-24T10:27:30.607 Function completed (Success, Id=e304450c-ff48-44dc-ba2e-1df7209a9d22)


<!-- URLs. -->

[Portal do Azure]: https://portal.azure.com

<!---HONumber=AcomDC_0921_2016-->
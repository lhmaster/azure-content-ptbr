<properties
    pageTitle="Referência da API de relatório de atividade de entrada do Azure Active Directory | Microsoft Azure"
    description="Referência para a API de relatório de atividade de entrada do Azure Active Directory"
    services="active-directory"
    documentationCenter=""
    authors="dhanyahk"
    manager="femila"
    editor=""/>

<tags
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="09/25/2016"
    ms.author="dhanyahk;markvi"/>

# Referência da API de relatório de atividade de entrada do Azure Active Directory


Este tópico faz parte de uma coleção de tópicos sobre a API de relatório do Azure Active Directory. Os relatórios do Azure AD fornecem uma API que permite a você acessar dados de relatórios de atividade de entrada usando código ou ferramentas relacionadas. O escopo deste tópico é fornecer informações de referência sobre a **API de relatório de atividade de entrada**.

Consulte:

- [Atividades de entrada](active-directory-reporting-azure-portal.md#sign-in-activities) para obter mais informações conceituais
- [Introdução à API de relatório do Azure Active Directory](active-directory-reporting-api-getting-started.md) para saber mais sobre a API de relatório.

Para dúvidas, problemas ou comentários, entre em contato com a [Ajuda de relatório do AAD](mailto:aadreportinghelp@microsoft.com).



## Quem pode acessar os dados da API?

- Usuários na função de Administrador de segurança ou Leitor de segurança

- Administradores globais

- Qualquer aplicativo que tenha autorização para acessar a API (a autorização do aplicativo pode ser definida somente com base na permissão de Administrador Global)



## Pré-requisitos

Para acessar esse relatório por meio da API de relatórios, você deve ter:

- Um [Azure Active Directory Premium edição P1 ou P2](active-directory-editions.md)

- Concluído os [pré-requisitos para acessar a API de relatório do Azure AD](active-directory-reporting-api-prerequisites.md).


##Como acessar a API

Você pode acessar essa API por meio de [Graph Explorer](https://graphexplorer2.cloudapp.net) ou usando programaticamente, por exemplo, o PowerShell. Para o PowerShell interpretar corretamente a sintaxe de filtro OData usada nas chamadas REST Graph do AAD, você deve usar o caractere de acento grave para "escapar" o caractere $. O caractere de acento grave serve como [caractere de escape do PowerShell](https://technet.microsoft.com/library/hh847755.aspx), permitindo que o PowerShell faça uma interpretação literal do caractere $ e evite confundi-lo como um nome de variável do PowerShell (ou seja: $filter).

O foco deste tópico é no Graph Explorer. Para obter um exemplo do PowerShell, consulte [scripts do PowerShell](active-directory-reporting-api-sign-in-activity-samples.md#powershell-script).


## Ponto de extremidade de API

Você pode acessar essa API usando o seguinte URI de base:
	
	https://graph.windows.net/contoso.com/activities/signinEvents?api-version=beta  



Devido ao volume de dados, essa API tem um limite de um milhão de registros retornados.

Essa chamada retorna os dados em lotes. Cada lote tem no máximo 1000 registros. Para obter o próximo lote de registros, use o link Próximo. Obtenha as informações de [skiptoken](https://msdn.microsoft.com/library/dd942121.aspx) do primeiro conjunto de registros retornados. O token skip estará no final do conjunto de resultados.

	https://graph.windows.net/$tenantdomain/activities/signinEvents?api-version=beta&%24skiptoken=-1339686058


## Filtros com suporte

Você pode reduzir o número de registros retornados por uma chamada de API na forma de um filtro. Para entrar dados de entrada relacionados à API, os filtros a seguir recebem suporte:

- **$top = <número de registros a serem retornados>** – Para limitar o número de registros retornados. Esta é uma operação cara. Não use esse filtro se você quiser retornar milhares de objetos.
- **$filter = <sua instrução de filtro>** – Para especificar, com base nos campos de filtro com suporte, o tipo de registro com o qual você se preocupa



## Campos de filtro e operadores com suporte

Para especificar o tipo de registro com o qual você se preocupa, compile uma instrução de filtro que possa conter um ou uma combinação dos campos de filtro a seguir:

- [signinDateTime](#signindatetime) – Define uma data ou intervalo de datas

- [userId](#userid) – Define um usuário específico com base na ID do usuário.

- [userPrincipalName](#userprincipalname) – Define um usuário específico com base no UPN (nome principal do usuário) do usuário

- [appId](#appid) – Define um aplicativo específico com base na ID do aplicativo

- [appDisplayName](#appdisplayname) – Define um aplicativo específico com base no nome para exibição do aplicativo

- [loginStatus](#loginStatus) – Define o status dos logons (sucesso/falha)


> [AZURE.NOTE] Ao usar o Graph Explorer, você precisa usar a capitalização correta para cada letra em seus campos de filtro.


Para restringir o escopo dos dados retornados, você pode compilar combinações dos campos de filtro e filtros com suporte. Por exemplo, a instrução a seguir retorna os 10 primeiros registros entre 1º de julho de 2016 e 6 de julho de 2016:

	https://graph.windows.net/contoso.com/activities/signinEvents?api-version=beta&$top=10&$filter=signinDateTime+ge+2016-07-01T17:05:21Z+and+signinDateTime+le+2016-07-07T00:00:00Z


----------

### signinDateTime

**Operadores com suporte**: eq, ge, le, gt, lt

**Exemplo**:

Como usar uma data específica

	$filter=signinDateTime+eq+2016-04-25T23:59:00Z	



Como usar um intervalo de datas

	$filter=signinDateTime+ge+2016-07-01T17:05:21Z+and+signinDateTime+le+2016-07-07T17:05:21Z


**Observações**:

O parâmetro datetime deve estar no formato UTC


----------

### coluna

**Operadores com suporte**: eq

**Exemplo**:

	$filter=userId+eq+’00000000-0000-0000-0000-000000000000’

**Observações**:

O valor de userID é um valor de cadeia de caracteres



----------

### userPrincipalName

**Operadores com suporte**: eq

**Exemplo**:

	$filter=userPrincipalName+eq+'audrey.oliver@wingtiptoysonline.com' 


**Observações**:

O valor de userPrincipalName é um valor de cadeia de caracteres

----------

### appId

**Operadores com suporte**: eq

**Exemplo**:

	$filter=appId+eq+’00000000-0000-0000-0000-000000000000’



**Observações**:

O valor de appId é um valor de cadeia de caracteres

----------


### appDisplayName

**Operadores com suporte**: eq

**Exemplo**:

	$filter=appDisplayName+eq+'Azure+Portal' 


**Observações**:

O valor de appDisplayName é um valor de cadeia de caracteres

----------

### loginStatus

**Operadores com suporte**: eq

**Exemplo**:

	$filter=loginStatus+eq+'1'  


**Observações**:

Há duas opções para o loginStatus: 0 – êxito, 1 – Falha

----------



## Próximas etapas

- Quer ver exemplos para atividades de entrada filtradas? Confira [Exemplos de API de relatório de atividade de entrada do Azure Active Directory](active-directory-reporting-api-sign-in-activity-samples.md).

- Quer saber mais sobre a API de relatório do Azure AD? Confira [Introdução à API de relatório do Azure Active Directory](active-directory-reporting-api-getting-started.md).

<!---HONumber=AcomDC_0928_2016-->
<properties
	pageTitle="Vários domínios do Azure AD Connect"
	description="Este documento descreve a instalação e a configuração de vários domínios de nível superior com o O365 e o Azure AD."
	services="active-directory"
	documentationCenter=""
	authors="billmath"
	manager="femila"
	editor="curtand"/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/08/2016"
	ms.author="billmath"/>

# Suporte a Vários Domínios para Federação com o Azure AD
A documentação a seguir fornece orientação sobre como usar vários domínios e subdomínios de nível superior ao estabelecer uma federação com o Office 365 ou com domínios do Azure AD.

## Suporte para vários domínios de nível superior
A federação de vários domínios de nível superior com o Azure AD requer configuração adicional que não é necessária na federação de um único domínio de nível superior.

Quando um domínio é federado com o Azure AD, várias propriedades são definidas no domínio no Azure. Uma delas é IssuerUri. Isso é um URI usado pelo Azure AD para identificar o domínio ao qual o token está associado. O URI não precisa resolver para nada, mas deve ser um URI válido. Por padrão, o Azure AD define isso como o valor do identificador do serviço de federação em sua configuração local do AD FS.

>[AZURE.NOTE]O identificador do serviço de federação é um URI que identifica exclusivamente um serviço de federação. O serviço de federação é uma instância do AD FS que funciona como o serviço de token de segurança.

Você pode exibir o IssuerUri usando o comando `Get-MsolDomainFederationSettings - DomainName <your domain>` do PowerShell.

![Get-MsolDomainFederationSettings](./media/active-directory-multiple-domains/MsolDomainFederationSettings.png)

Um problema surge quando queremos adicionar mais de um domínio de nível superior. Por exemplo, digamos que você tenha configurado a federação entre o Azure AD e seu ambiente local. Para este documento, estou usando bmcontoso.com. Agora adicionei um segundo domínio de nível superior, bmfabrikam.com.

![Domínios](./media/active-directory-multiple-domains/domains.png)

Quando tentamos converter nosso domínio bmfabrikam.com em federado, recebemos um erro. A razão para isso é que o Azure AD tem uma restrição que não permite que a propriedade IssuerUri tenha o mesmo valor para mais de um domínio.
  

![Erro de federação](./media/active-directory-multiple-domains/error.png)

### Parâmetro SupportMultipleDomain

Para solucionar isso, precisamos adicionar um IssuerUri diferente. Isso pode ser feito usando o parâmetro `-SupportMultipleDomain`. Esse parâmetro é usado com os seguintes cmdlets:
	
- `New-MsolFederatedDomain`
- `Convert-MsolDomaintoFederated`
- `Update-MsolFederatedDomain`

Esse parâmetro faz o Azure AD configurar o IssuerUri para se basear no nome do domínio. Ele será exclusivo nos diretórios no Azure AD. O uso do parâmetro permite que o comando do PowerShell seja concluído com êxito.

![Erro de federação](./media/active-directory-multiple-domains/convert.png)

Observando as configurações do novo domínio bmfabrikam.com, você pode ver o seguinte:

![Erro de federação](./media/active-directory-multiple-domains/settings.png)

Observe que `-SupportMultipleDomain` não altera os outros pontos de extremidade que ainda estão configurados para apontar para o serviço de federação em adfs.bmcontoso.com.

O `-SupportMultipleDomain` também garante que o sistema de AD FS inclua o valor de emissor adequado em tokens emitidos para o Azure AD. Ele faz isso usando a parte do domínio do UPN dos usuários e configurando isso como o domínio no issuerURI, ou seja, https://{upn suffix}/adfs/services/trust.

Dessa forma, durante a autenticação no Azure AD ou Office 365, o elemento IssuerURI no token do usuário é usado para localizar o domínio no Azure AD. Se não houver correspondência, a autenticação falhará.

Por exemplo, se a UPN de um usuário for bsimon@bmcontoso.com, o elemento IssuerURI nas emissões de token do AD FS será definido como http://bmcontoso.com/adfs/services/trust. Isso corresponderá à configuração do Azure AD, e a autenticação será bem-sucedida.

Abaixo vemos a regra de declaração personalizada que implementa essa lógica:

    c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type =   "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, ".+@(?<domain>.+)", "http://${domain}/adfs/services/trust/"));


>[AZURE.IMPORTANT]Para usar a opção - SupportMultipleDomain quando tentar adicionar um novo domínio ou converter domínios já adicionados, você precisará ter sua confiança federada configurada para dar suporte a eles desde o início.


## Como atualizar a relação de confiança entre o AD FS e o Azure AD
Se você não tiver configurado a confiança federada entre o AD FS e a instância do Azure AD, será preciso recriá-la. Isso ocorre porque, quando ela é originalmente configurada sem o parâmetro `-SupportMultipleDomain`, o IssuerUri é definido com o valor padrão. Na captura de tela abaixo, você pode ver que o IssuerUri está definido como https://adfs.bmcontoso.com/adfs/services/trust.

Agora, se adicionarmos um novo domínio no portal do Azure AD com êxito e tentarmos convertê-lo usando `Convert-MsolDomaintoFederated -DomainName <your domain>`, obteremos o seguinte erro.

![Erro de federação](./media/active-directory-multiple-domains/trust1.png)

Se você tentar adicionar a opção `-SupportMultipleDomain`, receberá o seguinte erro:

![Erro de federação](./media/active-directory-multiple-domains/trust2.png)

A simples tentativa de executar `Update-MsolFederatedDomain -DomainName <your domain> -SupportMultipleDomain` no domínio original também resultará em erro.

![Erro de federação](./media/active-directory-multiple-domains/trust3.png)

Use as etapas a seguir para adicionar mais um domínio de nível superior. Se você já tiver adicionado um domínio e não usou o parâmetro `-SupportMultipleDomain`, utilize as etapas para remover e atualizar o domínio original. Se você ainda não tiver adicionado um domínio de nível superior, poderá começar com as etapas para adicionar um domínio usando o PowerShell do Azure AD Connect.

Use as etapas a seguir para remover a relação de confiança do Microsoft Online e atualizar seu domínio original.

2.  No servidor de Federação do AD FS, abra **Gerenciamento do AD FS.**
2.  À esquerda, expanda **Relações de Confiança** e **Confianças de Terceira Parte Confiável**
3.  À direita, exclua a entrada **Plataforma de Identidade do Microsoft Office 365**. ![Remover o Microsoft Online](./media/active-directory-multiple-domains/trust4.png)
1.  Em um computador que tenha o [Módulo do Azure Active Directory para Windows PowerShell](https://msdn.microsoft.com/library/azure/jj151815.aspx) instalado, execute o seguinte: `$cred=Get-Credential`.
2.  Insira o nome de usuário e a senha de um administrador global para o domínio do Azure AD com o qual você está federando
2.  No PowerShell, digite `Connect-MsolService -Credential $cred`
4.  No PowerShell, digite `Update-MSOLFederatedDomain -DomainName <Federated Domain Name> -SupportMultipleDomain`. Isso é para o domínio original. Usando os domínios acima, seria: `Update-MsolFederatedDomain -DomainName bmcontoso.com -SupportMultipleDomain`


Use as etapas a seguir para adicionar o novo domínio de nível superior usando o PowerShell

1.  Em um computador que tenha o [Módulo do Azure Active Directory para Windows PowerShell](https://msdn.microsoft.com/library/azure/jj151815.aspx) instalado, execute o seguinte: `$cred=Get-Credential`.
2.  Insira o nome de usuário e a senha de um administrador global para o domínio do Azure AD com o qual você está federando
2.  No PowerShell, digite `Connect-MsolService -Credential $cred`
3.  No PowerShell, digite `New-MsolFederatedDomain –SupportMultipleDomain –DomainName`

Use as etapas a seguir para adicionar o novo domínio de nível superior usando o Azure AD Connect.

1.	Inicie o Azure AD Connect na área de trabalho ou no menu Iniciar
2.	Escolha "Adicionar um domínio do Azure AD adicional" ![Adicionar um domínio do Azure AD adicional](./media/active-directory-multiple-domains/add1.png)
3.	Insira as credenciais do Azure AD e do Active Directory
4.	Escolha o segundo domínio que você deseja configurar para federação. ![Adicionar um domínio do Azure AD adicional](./media/active-directory-multiple-domains/add2.png)
5.	Clique em Instalar


### Verifique o novo domínio de nível superior
Usando o comando `Get-MsolDomainFederationSettings - DomainName <your domain>` do PowerShell, você pode exibir o IssuerUri atualizado. A captura de tela abaixo mostra que as configurações de federação foram atualizadas no domínio original http://bmcontoso.com/adfs/services/trust

![Get-MsolDomainFederationSettings](./media/active-directory-multiple-domains/MsolDomainFederationSettings.png)

E o IssuerUri em nosso novo domínio foi definido como https://bmfabrikam.com/adfs/services/trust

![Get-MsolDomainFederationSettings](./media/active-directory-multiple-domains/settings2.png)


##Suporte para Subdomínios
Quando você adiciona um subdomínio, devido à maneira usada pelo Azure AD para tratar domínios, ele herda as configurações do pai. Isso significa que o IssuerUri precisa corresponder aos pais.

Digamos, por exemplo, que tenho bmcontoso.com e adiciono corp.bmcontoso.com. Isso significa que o IssuerUri de um usuário corp.bmcontoso.com precisará ser **http://bmcontoso.com/adfs/services/trust.** No entanto, a regra padrão implementada acima para o Azure AD gerará um token com um emissor como **http://corp.bmcontoso.com/adfs/services/trust.**, que não corresponderá ao domínio do valor obrigatório e fará com que a autenticação falhe.

### Como habilitar o suporte para subdomínios
Para solucionar esse problema, a relação de confiança de terceira parte confiável do AD FS do Microsoft Online precisa ser atualizada. Para isso, você precisa configurar a regra de declaração personalizada para que ela ignore os subdomínios do sufixo UPN do usuário ao construir o valor de Issuer personalizado.

A declaração a seguir fará isso:

    c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, "^((.*)([.|@]))?(?<domain>[^.]*[.].*)$", "http://${domain}/adfs/services/trust/"));

Use as etapas a seguir para adicionar uma declaração personalizada que dê suporte a subdomínios.

1.	Abra o gerenciamento do AD FS
2.	Clique com botão direito do mouse na relação de confiança de terceira parte confiável do Microsoft Online e escolha Editar regras de declaração
3.	Escolha a terceira regra de declaração e substitua ![Editar declaração](./media/active-directory-multiple-domains/sub1.png)
4.	Substitua a declaração atual:
    
	    c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, ".+@(?<domain>.+)","http://${domain}/adfs/services/trust/"));
    	
	por:
    
	    `c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, "^((.*)([.|@]))?(?<domain>[^.]*[.].*)$", "http://${domain}/adfs/services/trust/"));`
	
![Substituir declaração](./media/active-directory-multiple-domains/sub2.png)
5.	Clique em OK. Clique em Aplicar. Clique em OK. Feche o gerenciamento do AD FS.

<!---HONumber=AcomDC_0928_2016-->
<properties
   pageTitle="Modelo de armazenamento do Gerenciador de Recursos | Microsoft Azure"
   description="Mostra o esquema do Gerenciador de Recursos para a implantação de contas de armazenamento por meio de um modelo."
   services="azure-resource-manager,storage"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="04/05/2016"
   ms.author="tomfitz"/>

# Conta de armazenamento - esquema do modelo

Cria uma conta de armazenamento.

## Formato de esquema

Para criar uma conta de armazenamento, adicione o esquema a seguir à seção de recursos do seu modelo.

    {
        "type": "Microsoft.Storage/storageAccounts",
        "apiVersion": "2015-06-15",
        "name": string,
        "location": string,
        "properties": 
        {
        	"accountType": string
        }
    }

## Valores

As tabelas a seguir descrevem os valores necessários para definir no esquema.

| Nome | Valor |
| ---- | ---- |
| type | Enum<br />Obrigatório<br />**Microsoft.Storage/storageAccounts**<br /><br />O tipo de recurso a ser criado. |
| apiVersion | Enum<br />Obrigatório<br />**2015-06-15** ou **2015-05-01-preview**<br /><br />A versão da API a ser usada para criar o recurso. | 
| name | String<br />Obrigatório<br />Entre 3 e 24 caracteres, somente números e letras minúsculas.<br /><br />O nome da conta de armazenamento a ser criada. O nome deve ser exclusivo em todo o Azure. Considere o uso da função [uniqueString](resource-group-template-functions.md#uniquestring) com a convenção de nomenclatura, como mostrado no exemplo abaixo. |
| location | String<br />Obrigatório<br />Uma região que dá suporte a contas de armazenamento. Para determinar as regiões válidas, veja as [regiões com suporte](resource-manager-supported-services.md#supported-regions).<br /><br />A região que hospedará a conta de armazenamento. |
| propriedades | Object<br />Obrigatório<br />[properties object](#properties)<br /><br />Um objeto que especifica o tipo de conta de armazenamento a ser criada. |

<a id="properties" />
### properties object

| Nome | Valor |
| ---- | ---- | 
| accountType | String<br />Obrigatório<br />**Standard\_LRS**, **Standard\_ZRS**, **Standard\_GRS**, **Standard\_RAGRS** ou **Premium\_LRS**<br /><br />O tipo de conta de armazenamento. Os valores permitidos correspondem ao armazenamento com Redundância Local Standard, armazenamento com Redundância de Zona Standard, armazenamento com Redundância Geográfica Standard, armazenamento com Redundância Geográfica com Acesso de Leitura Standard e armazenamento com Redundância Local Premium. Para obter informações sobre esses tipos de conta, veja [Replicação do Armazenamento do Azure](./storage/storage-redundancy.md). |

	
## Exemplos

O exemplo a seguir implementa uma conta de armazenamento com Redundância Local Standard com um nome exclusivo baseado na ID de grupo de recursos.

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {},
        "resources": [
            {
                "type": "Microsoft.Storage/storageAccounts",
                "apiVersion": "2015-06-15",
                "name": "[concat('storage', uniqueString(resourceGroup().id))]",
                "location": "[resourceGroup().location]",
                "properties": 
        	{
                    "accountType": "Standard_LRS"
        	}
            }
        ],
        "outputs": {}
    }

## Modelos de início rápido

Há muitos modelos de início rápido que incluem uma conta de armazenamento. Os modelos a seguir ilustram alguns cenários comuns:

- [Criar uma conta de armazenamento padrão](https://azure.microsoft.com/documentation/templates/101-storage-account-create)
- [Implantação simples de uma VM do Windows](https://azure.microsoft.com/documentation/templates/101-vm-simple-windows)
- [Implantação simples de uma VM do Linux](https://azure.microsoft.com/documentation/templates/101-vm-simple-linux)
- [Criar um perfil de CDN, um ponto de extremidade CDN com uma conta de armazenamento como origem](https://azure.microsoft.com/documentation/templates/201-cdn-with-storage-account)
- [Criar um farm de SharePoint de alta disponibilidade com 9 VMs usando a extensão de DSC do Powershell](https://azure.microsoft.com/documentation/templates/sharepoint-server-farm-ha)
- [Implantação simples de um cluster seguro do Service Fabric de 5 nós com WAD habilitado](https://azure.microsoft.com/documentation/templates/service-fabric-secure-cluster-5-node-1-nodetype-wad)
- [Criar uma máquina virtual de uma imagem do Windows com 4 discos de dados vazios](https://azure.microsoft.com/documentation/templates/101-vm-multiple-data-disk)


## Próximas etapas

- Para obter informações gerais sobre armazenamento, veja [Introdução ao Armazenamento do Microsoft Azure](./storage/storage-introduction.md).
- Por ver exemplos de modelos que usam uma nova conta de armazenamento com uma Máquina Virtual, veja [Implantar uma VM simples do Linux](https://azure.microsoft.com/documentation/templates/101-simple-linux-vm/) ou [Implantar uma VM simples do Windows](https://azure.microsoft.com/documentation/templates/101-simple-windows-vm/).

<!---HONumber=AcomDC_0406_2016-->
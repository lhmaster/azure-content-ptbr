<properties
	pageTitle="Capturar uma imagem de uma VM do Linux | Microsoft Azure"
	description="Saiba como capturar uma imagem de uma VM (máquina virtual) do Azure baseada em Linux criada com o modelo de implantação clássico."
	services="virtual-machines-linux"
	documentationCenter=""
	authors="iainfoulds"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/31/2016"
	ms.author="iainfou"/>


# Como capturar uma máquina virtual clássica do Linux como uma imagem

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)] Saiba como [executar estas etapas usando o modelo do Resource Manager](virtual-machines-linux-capture-image.md).

Este artigo mostra como capturar uma máquina virtual do Azure clássica executando o Linux como uma imagem para criar outras máquinas virtuais. Esta imagem inclui o disco do sistema operacional e discos de dados anexados à máquina virtual. Ele não inclui a configuração de rede, então você precisará configurá-la quando criar as outras máquinas virtuais por meio da imagem.

O Azure armazena a imagem em **Imagens**, juntamente com quaisquer imagens carregadas. Para saber mais sobre imagens, confira [Sobre imagens da máquina virtual no Azure][].

## Antes de começar

Essas etapas pressupõem que você já criou uma máquina virtual do Azure usando o modelo de implantação clássico e configurou o sistema operacional, incluindo a anexação dos discos de dados. Se você precisar criar uma VM, leia [Como criar uma máquina virtual do Linux][].


## Capturar a máquina virtual

1. [Conecte-se à máquina virtual](virtual-machines-linux-mac-create-ssh-keys.md) usando um cliente SSH de sua escolha.

2. Na janela SSH, digite o comando a seguir. A saída de `waagent` pode variar um pouco dependendo da versão do utilitário:

	`sudo waagent -deprovision+user`

	Esse comando tenta limpar o sistema e torná-lo adequado para o reprovisionamento. Essa operação realiza as seguintes tarefas:

	- Remove as chaves de host SSH (se Provisioning.RegenerateSshHostKeyPair for 'y' no arquivo de configuração)
	- Limpa a configuração de servidor de nomes em /etc/resolv.conf
	- Remove a senha do usuário `root` de /etc/shadow (se Provisioning.DeleteRootPassword for 'y' no arquivo de configuração)
	- Remove concessões de cliente DHCP em cache
	- Reinicia o nome de host para localdomain.localdomain
	- Exclui a última conta de usuário provisionado (obtida em /var/lib/waagent) **e dados associados**.

	>[AZURE.NOTE] O desprovisionamento exclui arquivos e dados para “generalizar” a imagem. Execute este comando apenas em uma máquina virtual que deseje capturar como um novo modelo de imagem. Ele não garante que a imagem esteja sem nenhuma informação confidencial ou seja adequada para redistribuição a terceiros.


3. Digite **y** para continuar. Você pode adicionar o parâmetro `-force` para evitar esta etapa de confirmação.

4. Digite **Exit** para fechar o cliente SSH.

	>[AZURE.NOTE] As etapas restantes presumem que você já [instalou a CLI do Azure](../xplat-cli-install.md) no computador cliente. Todas as etapas a seguir também podem ser executadas no [Portal Clássico do Azure][].

5. No computador cliente, abra a CLI do Azure e faça logon com sua assinatura do Azure. Para obter detalhes, leia [Conectar-se a uma assinatura do Azure da CLI do Azure](../xplat-cli-connect.md).

6. Verifique se você está no modo de Gerenciamento de Serviços:

	`azure config mode asm`

7. Desligue a máquina virtual que já foi desprovisionada nas etapas anteriores com:

	`azure vm shutdown <your-virtual-machine-name>`

	>[AZURE.NOTE] É possível encontrar todas as máquinas virtuais criadas em sua assinatura usando `azure vm list`

8. Quando a máquina virtual for interrompida, capture a imagem com o comando:

	`azure vm capture -t <your-virtual-machine-name> <new-image-name>`

	Digite o nome da imagem que você deseja substituir no lugar de _new-image-name_. Este comando cria uma imagem generalizada do SO. O subcomando `-t` exclui a máquina virtual original.

9.	A nova imagem agora está disponível na lista de imagens que podem ser usadas para configurar qualquer nova máquina virtual. Você pode exibi-la com o comando:

	`azure vm image list`

	No [Portal Clássico do Azure][], ela será exibida na lista **IMAGENS**.

	![Captura de imagem bem-sucedida](./media/virtual-machines-linux-classic-capture-image/VMCapturedImageAvailable.png)


## Próximas etapas
A imagem está pronta para ser usada para criar máquinas virtuais. É possível usar o comando `azure vm create` da CLI do Azure e fornecer o nome da imagem que você criou. Consulte [Como usar a CLI do Azure com o modelo de implantação Clássica](../virtual-machines-command-line-tools.md) para obter detalhes sobre o comando. Como alternativa, use o [Portal Clássico do Azure][] para criar uma máquina virtual personalizada usando o método **Da galeria** e selecionando a imagem que você criou. Consulte [Como criar uma máquina virtual personalizada][] para obter mais detalhes.

**Consulte também:** [Guia do usuário do agente Linux para o Azure](virtual-machines-linux-agent-user-guide.md)

[Portal Clássico do Azure]: http://manage.windowsazure.com
[Sobre imagens da máquina virtual no Azure]: virtual-machines-linux-classic-about-images.md
[Como criar uma máquina virtual personalizada]: virtual-machines-linux-classic-create-custom.md
[How to Attach a Data Disk to a Virtual Machine]: virtual-machines-windows-classic-attach-disk.md
[Como criar uma máquina virtual do Linux]: virtual-machines-linux-classic-create-custom.md

<!---HONumber=AcomDC_0907_2016-->
<properties 
	pageTitle="Configurar o LVM de software em uma máquina virtual que executa o Linux | Microsoft Azure" 
	description="Saiba como configurar o LVM no Linux no Azure." 
	services="virtual-machines-linux" 
	documentationCenter="na" 
	authors="szarkos"  
	manager="timlt" 
	editor="tysonn"
	tag="azure-service-management,azure-resource-manager" />

<tags 
	ms.service="virtual-machines-linux" 
	ms.workload="infrastructure-services" 
	ms.tgt_pltfrm="vm-linux" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="08/24/2016" 
	ms.author="szark"/>


# Configurar o LVM em uma VM Linux no Azure

Este documento discutirá como configurar o LVM (Gerenciador de Volume Lógico) em sua máquina virtual do Azure. Embora seja possível configurar o LVM em qualquer disco anexado à máquina virtual, por padrão a maioria das imagens de nuvem não terá o LVM configurado no disco do sistema operacional. Isso é para evitar problemas com grupos de volumes duplicados se o disco do sistema operacional for anexado a outra VM do mesmo tipo e distribuição, ou seja, durante um cenário de recuperação. Portanto, é recomendável usar o LVM somente em discos de dados.


## Volumes lógicos lineares versus lógicos distribuídos

O LVM pode ser usado para combinar um número de discos físicos em um único volume de armazenamento. Por padrão, geralmente o LVM cria volumes lógicos lineares, o que significa que o armazenamento físico é concatenado. Nesse caso, as operações de leitura/gravação serão normalmente enviadas para um único disco. Por outro lado, também podemos criar volumes lógicos distribuídos onde as leituras e as gravações são distribuídas por vários discos contidos no grupo de volumes (isto é, semelhante ao RAID0). Por motivos de desempenho, é provável que você queira distribuir seus volumes lógicos para que as leituras e as gravações utilizem todos os discos de dados anexados.

Este documento descreve como combinar diversos discos de dados em um único grupo de volumes e, em seguida, criar um volume lógico distribuído. As etapas a seguir são um pouco generalizadas para funcionar com a maioria das distribuições. Na maioria dos casos, os utilitários e os fluxos de trabalho para o gerenciamento de LVM no Azure não são fundamentalmente diferentes de outros ambientes. Como de costume, também consulte o fornecedor do Linux para obter documentação e práticas recomendadas para usar o LVM com sua distribuição específica.


## Anexando discos de dados
Geralmente, as pessoas desejam começar com dois ou mais discos de dados vazios ao usar LVM. Com base em suas necessidades de E/S, você pode optar por anexar discos que serão armazenados em nosso armazenamento padrão, com até 500 E/S/s por disco, ou em nosso armazenamento Premium, com até 5.000 E/S/s por disco. Este artigo não se aprofundará em detalhes sobre como provisionar e anexar discos de dados para uma máquina virtual Linux. Confira o artigo [anexar um disco](virtual-machines-linux-add-disk.md) do Microsoft Azure para obter instruções detalhadas sobre como anexar um disco de dados vazio a uma máquina virtual do Linux no Azure.

## Instalar os utilitários do LVM

- **Ubuntu**

		# sudo apt-get update
		# sudo apt-get install lvm2

- **RHEL, CentOS e Oracle Linux**

		# sudo yum install lvm2

- **SLES 12 e openSUSE**

		# sudo zypper install lvm2

- **SLES 11**

		# sudo zypper install lvm2

	No SLES11, também é necessário editar /etc/sysconfig/lvm e definir `LVM_ACTIVATED_ON_DISCOVERED` como "enable":

		LVM_ACTIVATED_ON_DISCOVERED="enable" 


## Configurar o LVM
Neste guia, supomos que você tem três discos de dados anexados, que chamaremos de `/dev/sdc`, `/dev/sdd` e `/dev/sde`. Observe que eles nem sempre poderão ter os mesmos nomes de caminho na sua VM. Você pode executar o '`sudo fdisk -l`' ou um comando semelhante para listar os discos disponíveis.

1. Prepare os volumes físicos:

		# sudo pvcreate /dev/sd[cde]
		  Physical volume "/dev/sdc" successfully created
		  Physical volume "/dev/sdd" successfully created
		  Physical volume "/dev/sde" successfully created


2.  Crie um grupo de volumes. Neste exemplo, chamamos o grupo de volumes de "data-vg01":

		# sudo vgcreate data-vg01 /dev/sd[cde]
		  Volume group "data-vg01" successfully created


3. Crie os volumes lógicos. O comando abaixo criará um único volume lógico chamado "data-lv01" para abranger o grupo de volumes completo, mas observe que também é possível criar vários volumes lógicos no grupo de volumes.

		# sudo lvcreate --extents 100%FREE --stripes 3 --name data-lv01 data-vg01
		  Logical volume "data-lv01" created.


4. Formatar o volume lógico

		# sudo mkfs -t ext4 /dev/data-vg01/data-lv01

  >[AZURE.NOTE] Com o SLES11, use "-t ext3" em vez de ext4. O SLES11 só oferece suporte a acesso somente leitura para sistemas de arquivos ext4.


## Adicionar o novo sistema de arquivos a /etc/fstab

**Cuidado:** a edição inadequada do arquivo /etc/fstab pode resultar em um sistema não inicializável. Se não tiver certeza, consulte a documentação de distribuição para obter informações sobre como editá-lo corretamente. Também é recomendável que um backup do arquivo /etc/fstab seja criado antes da edição.

1. Crie o ponto de montagem desejado para o novo sistema de arquivos, por exemplo:

		# sudo mkdir /data


2. Localizar o caminho do volume lógico

		# lvdisplay
		--- Logical volume ---
		LV Path                /dev/data-vg01/data-lv01
		....


3. Abra o /etc/fstab em um editor de texto e adicione uma entrada para o novo sistema de arquivos, por exemplo:

		/dev/data-vg01/data-lv01  /data  ext4  defaults  0  2

	Em seguida, salve e feche o /etc/fstab.


4. Testar se a entrada /etc/fstab está correta:

		# sudo mount -a

	Se esse comando resultar em uma mensagem de erro, verifique a sintaxe no arquivo /etc/fstab.

	Em seguida, execute o comando `mount` para garantir que o sistema de arquivos esteja montado:

		# mount
		......
		/dev/mapper/data--vg01-data--lv01 on /data type ext4 (rw)


5. (Opcional) Parâmetros de inicialização à prova de falhas em /etc/fstab

	Várias distribuições incluem os parâmetros de montagem `nobootwait` ou `nofail` que podem ser adicionados ao arquivo /etc/fstab. Esses parâmetros permitem falhas ao montar um sistema de arquivos específico e permitem que o sistema Linux continue a inicialização, mesmo que não seja possível montar corretamente o sistema de arquivos RAID. Consulte a documentação da distribuição para obter mais informações sobre esses parâmetros.

	Exemplo (Ubuntu):

		/dev/data-vg01/data-lv01  /data  ext4  defaults,nobootwait  0  2

<!---HONumber=AcomDC_0831_2016-->
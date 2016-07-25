<properties
	pageTitle="Modelagem e exploração de dados com Spark | Microsoft Azure"
	description="Demonstra os recursos de exploração e modelagem de dados do kit de ferramentas do Spark MLlib."
	services="machine-learning"
	documentationCenter=""
	authors="bradsev,deguhath,gokuma"
	manager="paulettm"
	editor="cgronlun" />

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="06/14/2016"
	ms.author="deguhath;bradsev" />

# Modelagem e exploração de dados com Spark

[AZURE.INCLUDE [machine-learning-spark-modeling](../../includes/machine-learning-spark-modeling.md)]

Este passo a passo usa o HDInsight Spark para executar tarefas de exploração de dados e modelagem em um exemplo do conjunto de dados de corridas e tarifas de táxi de Nova York de 2013. Ele o orienta ao longo das etapas do [Processo de Ciência de Dados](http://aka.ms/datascienceprocess), de ponta a ponta, usando um cluster HDInsight Spark para processamento e blobs do Azure para armazenar os dados e os modelos. O processo explora e visualiza os dados transferidos de um Blob de Armazenamento do Azure e prepara os dados para criar modelos preditivos. Esses modelos são compilados usando o kit de ferramentas Spark MLlib para executar tarefas de classificação binária e modelagem de regressão.

- A tarefa de **classificação binária** consiste em prever se uma gorjeta é paga ou não pela corrida. 
- A tarefa de **regressão** consiste em prever o valor da gorjeta com base em outros recursos de gorjeta. 

Os modelos que usamos incluem regressão logística e linear, florestas aleatórias e árvores aumentadas gradientes:

- [Regressão linear com SGD](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.regression.LinearRegressionWithSGD) é um modelo de regressão linear que usa um método SGD (Stochastic Gradient Descent) para otimização e dimensionamento de recursos para prever os valores das gorjetas pagas. 
- [Regressão logística com LBFGS](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.classification.LogisticRegressionWithLBFGS) ou regressão "logit" é um modelo de regressão que pode ser usado quando a variável dependente é categórica para fazer a classificação de dados. LBFGS é um algoritmo de otimização quase Newton que aproxima o algoritmo BFGS (Broyden–Fletcher–Goldfarb–Shanno) usando uma quantidade limitada de memória do computador e que é amplamente usado no aprendizado de máquina.
- [Florestas aleatórias](http://spark.apache.org/docs/latest/mllib-ensembles.html#Random-Forests) são conjuntos de árvores de decisão. Elas combinam várias árvores de decisão para reduzir o risco de superajuste. As florestas aleatórias são usadas para regressão e classificação e podem lidar com recursos categóricos e estender-se à configuração de classificação multiclasse, não precisam de dimensionamento de recursos e podem capturar não linearidades e interações de recursos. As florestas aleatórias são um dos modelos de aprendizado de máquina com maior taxa de sucesso para classificação e regressão.
- [GBTs (árvores com aumento gradiente)](http://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) são conjuntos de árvores de decisão. As GBTs treinam árvores de decisão iterativamente para minimizar uma função de perda. As GBTs são usadas para regressão e classificação e podem lidar com recursos categóricos, não exigem o dimensionamento de recursos e podem capturar não linearidades e interações de recursos. Elas também podem ser usadas em uma configuração de classificação multiclasse.

As etapas de modelagem também contêm código que mostra como treinar, avaliar e salvar cada tipo de modelo. Python foi usado para codificar a solução e mostrar os gráficos relevantes.


>[AZURE.NOTE] Embora o kit de ferramentas Spark MLlib tenha sido projetado para trabalhar com grandes conjuntos de dados, para fins de demonstração de suas funcionalidades de modelagem, uma amostra relativamente pequena (cerca de 30 Mb usando 170 mil linhas, cerca de 0,1% do conjunto de dados original NYC) foi usada por conveniência. O exercício fornecido aqui é executado com eficiência em um cluster HDInsight com dois nós de trabalho (em aproximadamente 10 minutos). O mesmo código, com pequenas modificações, pode ser usado para processar conjuntos de dados maiores, com as modificações apropriadas para armazenar dados na memória em cache ou alterar o tamanho do cluster.

## Pré-requisitos

Você precisa de uma conta do Azure e um HDInsight Spark Você precisa de um cluster HDInsight 3.4 Spark 1.6 para concluir este passo a passo. Confira os requisitos na [Visão geral de ciência de dados usando o Spark no Azure HDInsight](machine-learning-data-science-spark-overview.md) para obter uma descrição dos dados NYC 2013 Taxi usados aqui e para obter instruções sobre como executar código de um bloco de anotações do Jupyter no cluster Spark. O bloco de anotações **machine-learning-data-science-spark-data-exploration-modeling.ipynb** que contém os exemplos de código deste tópico está disponível no [Github](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/pySpark).


[AZURE.INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]


## Instalação: locais de armazenamento, bibliotecas e o contexto predefinido do Spark

O Spark pode ler e gravar em um Blob de Armazenamento do Azure (também conhecido como WASB). Portanto, qualquer dado existente armazenado lá pode ser processado usando o Spark, e os resultados podem ser armazenados novamente no WASB.

Para salvar arquivos ou modelos no WASB, o caminho deve ser especificado corretamente. O contêiner padrão anexado ao cluster Spark pode ser referenciado usando um caminho que começa com: “wasb:///”. Outros locais são referenciados por "wasb://".


### Definir caminhos de diretório para locais de armazenamento no WASB

O exemplo de código a seguir especifica o local dos dados a serem lidos e o caminho do diretório de armazenamento de modelo em que a saída do modelo será salva.


	# SET PATHS TO FILE LOCATIONS: DATA AND MODEL STORAGE

	# LOCATION OF TRAINING DATA
	taxi_train_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Train.tsv";

	# SET THE MODEL STORAGE DIRECTORY PATH 
	# NOTE THAT THE FINAL BACKSLASH IN THE PATH IS NEEDED.
	modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/" 


### Importar bibliotecas

A instalação também requer a importação das bibliotecas necessárias. Defina o contexto do Spark e importe as bibliotecas necessárias com o código a seguir.


	# IMPORT LIBRARIES
	import pyspark
	from pyspark import SparkConf
	from pyspark import SparkContext
	from pyspark.sql import SQLContext
	import matplotlib
	import matplotlib.pyplot as plt
	from pyspark.sql import Row
	from pyspark.sql.functions import UserDefinedFunction
	from pyspark.sql.types import *
	import atexit
	from numpy import array
	import numpy as np
	import datetime


### Contexto predefinido do Spark e palavras mágicas do PySpark

Os kernels PySpark fornecidos com os notebooks do Jupyter têm um contexto predefinido e, portanto, não é necessário definir explicitamente contextos do Spark ou do Hive antes de começar a trabalhar com o aplicativo que está em desenvolvimento; eles estão disponíveis por padrão para você. Esses contextos são:

- sc - para o Spark 
- sqlContext - para o Hive

O kernel PySpark fornece algumas “palavras mágicas” predefinidas, que são comandos especiais que podem ser chamados com %%. Há dois comandos que são usados nesses exemplos de código.

- **%%local** Especificou que o código nas linhas posteriores será executado localmente. O código deve ser um código Python válido.
- **%%sql -o <variable name>** Executa uma consulta do Hive no sqlContext. Se o parâmetro -o for transmitido, o resultado da consulta será persistido no contexto %%local do Python como um quadro de dados do Pandas.
 

Para saber mais sobre os kernels para notebooks do Jupyter e as “palavras mágicas” predefinidas chamadas com %% (por exemplo, %%local) que eles fornecem, veja [Kernels disponíveis para notebooks do Jupyter com clusters HDInsight Spark Linux no HDInsight](../hdinsight/hdinsight-apache-spark-jupyter-notebook-kernels.md).
 

## Ingestão de dados de blob público

Esta seção contém o código para uma série de tarefas necessárias para ingerir a amostra de dados a ser modelada. Leia uma amostra de 0,1% do arquivo de corridas e tarifas de táxi associado (armazenado como um arquivo .tsv), formate e limpe os dados, armazene um quadro de dados em cache na memória e registre-o como uma tabela temporária no contexto do SQL.

A primeira etapa no processo de ciência de dados é ingerir os dados a serem analisados de fontes externas ou sistemas em que residem para seu ambiente de modelagem e exploração de dados. Esse ambiente é o Spark neste passo a passo. Esta seção contém o código para concluir uma série de tarefas:

- ingerir a amostra de dados a ser modelada
- ler o conjunto de dados (armazenado como um arquivo .tsv)
- formatar e limpar os dados
- criar e armazenar objetos em cache (RDDs ou quadros de dados) na memória
- registrá-lo como uma tabela temporária no contexto do SQL.

Aqui está o código para ingestão de dados.

	# INGEST DATA

	# RECORD START TIME
	timestart = datetime.datetime.now()

	# IMPORT FILE FROM PUBLIC BLOB
	taxi_train_file = sc.textFile(taxi_train_file_loc)
	
	# GET SCHEMA OF THE FILE FROM HEADER
	schema_string = taxi_train_file.first()
	fields = [StructField(field_name, StringType(), True) for field_name in schema_string.split('\t')]
	fields[7].dataType = IntegerType() #Pickup hour
	fields[8].dataType = IntegerType() # Pickup week
	fields[9].dataType = IntegerType() # Weekday
	fields[10].dataType = IntegerType() # Passenger count
	fields[11].dataType = FloatType() # Trip time in secs
	fields[12].dataType = FloatType() # Trip distance
	fields[19].dataType = FloatType() # Fare amount
	fields[20].dataType = FloatType() # Surcharge
	fields[21].dataType = FloatType() # Mta_tax
	fields[22].dataType = FloatType() # Tip amount
	fields[23].dataType = FloatType() # Tolls amount
	fields[24].dataType = FloatType() # Total amount
	fields[25].dataType = IntegerType() # Tipped or not
	fields[26].dataType = IntegerType() # Tip class
	taxi_schema = StructType(fields)
	
	# PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
	taxi_header = taxi_train_file.filter(lambda l: "medallion" in l)
	taxi_temp = taxi_train_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
	        .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
	                        float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
	                        float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))
	
	
	# CREATE DATA FRAME
	taxi_train_df = sqlContext.createDataFrame(taxi_temp, taxi_schema)
	
	# CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
	taxi_df_train_cleaned = taxi_train_df.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
	    .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
	    .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
	    .drop('direct_distance').drop('surcharge')\
	    .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )

	
	# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
	taxi_df_train_cleaned.cache()
	taxi_df_train_cleaned.count()
	
	# REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
	taxi_df_train_cleaned.registerTempTable("taxi_train")
	
	# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**SAÍDA:**

Tempo necessário para executar a célula acima: 51,72 segundos


## Visualização e exploração de dados

Depois que os dados forem incluídos no Spark, a próxima etapa no processo de ciência de dados será obter uma compreensão mais profunda dos dados por meio de exploração e visualização. Nesta seção, podemos examinar os dados de táxi usando consultas SQL e plotar as variáveis de destino e os recursos em potencial para inspeção visual. Especificamente, plotamos a frequência das contagens de passageiros em corridas de táxi, a frequência de gorjetas e como as gorjetas variam de acordo com o valor e o tipo de pagamento.

### Plotar um histograma de frequências de contagens de passageiros na amostra de corridas de táxi

Este código e os trechos de código posteriores usam as palavras mágicas do SQL para consultar a amostra e as palavras mágicas locais para plotar os dados.

- **Palavras mágicas do SQL (`%%sql`)** O kernel HDInsight PySpark dá suporte a consultas do HiveQL fáceis e embutidas no sqlContext. O argumento (-o VARIABLE\_NAME) persiste a saída da consulta SQL como um quadro de dados do Pandas no servidor do Jupyter. Isso significa que ele estará disponível no modo local.
- As **palavras mágicas do `%%local`** são usadas para executar o código localmente no servidor do Jupyter, que é o nó de cabeçalho do cluster HDInsight. Normalmente, você usa as palavras mágicas do `%%local` em conjunto com as palavras mágicas do `%%sql` com o parâmetro -o. O parâmetro -o persistiria a saída da consulta SQL localmente e, em seguida, as palavras mágicas de %%local disparariam o próximo conjunto de trechos de código para serem executados localmente na saída das consultas SQL que é persistida localmente

A saída será visualizada automaticamente após a execução do código.

Essa consulta recupera as corridas por contagem de passageiros.

	# PLOT FREQUENCY OF PASSENGER COUNTS IN TAXI TRIPS

	# HIVEQL QUERY AGAINST THE sqlContext
	%%sql -q -o sqlResults
	SELECT passenger_count, COUNT(*) as trip_counts 
	FROM taxi_train 
	WHERE passenger_count > 0 and passenger_count < 7 
	GROUP BY passenger_count 

Esse código cria um quadro de dados local da saída da consulta e plota os dados. As palavras mágicas do `%%local` criam um quadro de dados local, `sqlResults`, que pode ser usado para plotar com matplotlib.

>[AZURE.NOTE] Essas palavras mágicas do PySpark são usadas várias vezes neste passo a passo. Se a quantidade de dados for grande, você deverá obter uma amostra para criar um quadro de dados que se ajusta na memória local.

	#CREATE LOCAL DATA-FRAME AND USE FOR MATPLOTLIB PLOTTING

	# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
	%%local
	
	# USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES. 
	# CLICK ON THE TYPE OF PLOT TO BE GENERATED (E.G. LINE, AREA, BAR ETC.)
	sqlResults

Este é o código para plotar as corridas por contagens de passageiros

	# PLOT PASSENGER NUMBER VS. TRIP COUNTS
	%%local
	import matplotlib.pyplot as plt
	%matplotlib inline
	
	x_labels = sqlResults['passenger_count'].values
	fig = sqlResults[['trip_counts']].plot(kind='bar', facecolor='lightblue')
	fig.set_xticklabels(x_labels)
	fig.set_title('Counts of trips by passenger count')
	fig.set_xlabel('Passenger count in trips')
	fig.set_ylabel('Trip counts')
	plt.show()

**SAÍDA:**

![Frequência de corridas por contagem de passageiros](./media/machine-learning-data-science-spark-data-exploration-modeling/trip-freqency-by-passenger-count.png)

É possível selecionar entre vários tipos diferentes de visualizações (Tabela, Pizza, Linha, Área ou Barra) usando os botões de menu **Tipo** no notebook. A plotagem de Barras é mostrada aqui.
	
### Plote um histograma de valores de gorjetas e como o valor das gorjetas varia pelas tarifas e contagens de passageiros.

Use uma consulta SQL para obter amostra de dados.

	#PLOT HISTOGRAM OF TIP AMOUNTS AND VARIATION BY PASSENGER COUNT AND PAYMENT TYPE
	
	# HIVEQL QUERY AGAINST THE sqlContext
	%%sql -q -o sqlResults
	SELECT fare_amount, passenger_count, tip_amount, tipped 
	FROM taxi_train 
	WHERE passenger_count > 0 
	AND passenger_count < 7 
	AND fare_amount > 0 
	AND fare_amount < 200 
	AND payment_type in ('CSH', 'CRD') 
	AND tip_amount > 0 
	AND tip_amount < 25


Esta célula de código usa a consulta SQL para criar três plotagens dos dados.

	# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
	%%local
	
	# HISTOGRAM OF TIP AMOUNTS AND PASSENGER COUNT
	ax1 = sqlResults[['tip_amount']].plot(kind='hist', bins=25, facecolor='lightblue')
	ax1.set_title('Tip amount distribution')
	ax1.set_xlabel('Tip Amount ($)')
	ax1.set_ylabel('Counts')
	plt.suptitle('')
	plt.show()
	
	# TIP BY PASSENGER COUNT
	ax2 = sqlResults.boxplot(column=['tip_amount'], by=['passenger_count'])
	ax2.set_title('Tip amount by Passenger count')
	ax2.set_xlabel('Passenger count')
	ax2.set_ylabel('Tip Amount ($)')
	plt.suptitle('')
	plt.show()
	
	# TIP AMOUNT BY FARE AMOUNT, POINTS ARE SCALED BY PASSENGER COUNT
	ax = sqlResults.plot(kind='scatter', x= 'fare_amount', y = 'tip_amount', c='blue', alpha = 0.10, s=5*(sqlResults.passenger_count))
	ax.set_title('Tip amount by Fare amount')
	ax.set_xlabel('Fare Amount ($)')
	ax.set_ylabel('Tip Amount ($)')
	plt.axis([-2, 100, -2, 20])
	plt.show()


**SAÍDA:**

![Distribuição de valores de gorjetas](./media/machine-learning-data-science-spark-data-exploration-modeling/tip-amount-distribution.png)

![Valor de gorjeta por contagem de passageiros](./media/machine-learning-data-science-spark-data-exploration-modeling/tip-amount-by-passenger-count.png)

![Valor de gorjeta por valor de tarifa](./media/machine-learning-data-science-spark-data-exploration-modeling/tip-amount-by-fare-amount.png)


## Engenharia de recursos, transformação e preparação de dados para a modelagem
Esta seção descreve e fornece o código para os procedimentos usados para preparar dados para uso na modelagem ML. Ela mostra como realizar as seguintes tarefas:

- Criar um novo recurso reunindo horários em blocos de tempo de tráfego
- Indexar e codificar recursos categóricos
- Criar objetos de ponto rotulado para entrada em funções de ML
- Criar uma subamostragem aleatória dos dados e dividi-la em conjuntos de treinamento e teste
- Dimensionamento de recursos
- Armazenar objetos em cache na memória


### Criar um novo recurso reunindo horários em blocos de tempo de tráfego

Este código mostra como criar um novo recurso reunindo horários em blocos de tempo de tráfego e como armazenar em cache o quadro de dados resultante na memória. Quando RDDs (Conjuntos de Dados Resilientes Distribuídos) e quadros de dados são usados repetidamente, o armazenamento em cache leva a tempos de execução melhores. Da mesma forma, armazenamos em cache RDDs e quadros de dados em vários estágios no passo a passo.

	# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
	sqlStatement = """
	    SELECT *,
	    CASE
	     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
	     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
	     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
	     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
	    END as TrafficTimeBins
	    FROM taxi_train 
	"""
	taxi_df_train_with_newFeatures = sqlContext.sql(sqlStatement)
	
	# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
	# THE .COUNT() GOES THROUGH THE ENTIRE DATA-FRAME,
	# MATERIALIZES IT IN MEMORY, AND GIVES THE COUNT OF ROWS.
	taxi_df_train_with_newFeatures.cache()
	taxi_df_train_with_newFeatures.count()

**SAÍDA:**

126050

### Indexar e codificar recursos categóricos para entrada em funções de modelagem

Esta seção mostra como indexar ou codificar recursos categóricos para entrada nas funções de modelagem. As funções de modelagem e previsão de MLlib exigem que recursos com dados de entrada categóricos sejam indexados ou codificados antes do uso. Dependendo do modelo, você precisa indexá-los ou codificá-los de maneiras diferentes:

- A **Modelagem baseada em árvore** requer que as categorias sejam codificadas como valores numéricos (por exemplo, um recurso com três categorias pode ser codificado com 0, 1, 2). Isso é fornecido pela função [StringIndexer](http://spark.apache.org/docs/latest/ml-features.html#stringindexer) do MLlib. Essa função codifica uma coluna de cadeia de caracteres de rótulos para uma coluna de índices de rótulo que são ordenados por frequências de rótulos. Observe que, embora indexados com valores numéricos para entrada e manipulação de dados, os algoritmos baseados em árvore podem ser especificados para tratá-los adequadamente como categorias. 

- **Modelos de regressão linear e logística** exigem codificação one-hot, em que, por exemplo, um recurso com três categorias pode ser expandido em três colunas de recursos, em que cada uma contém 0 ou 1, dependendo da categoria de uma observação. O MLlib fornece a função [OneHotEncoder](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) para executar a codificação one-hot. Esse codificador mapeia uma coluna de índices de rótulo para uma coluna de vetores binários com, no máximo, um valor único. Essa codificação permite que os algoritmos que esperam recursos valiosos numéricos, como a regressão logística, sejam aplicados em recursos categóricos.

Aqui está o código para indexar e codificar recursos categóricos:


	# INDEX AND ENCODE CATEGORICAL FEATURES

	# RECORD START TIME
	timestart = datetime.datetime.now()

	# LOAD PYSPARK LIBRARIES	
	from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, VectorIndexer
	
	# INDEX AND ENCODE VENDOR_ID
	stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
	model = stringIndexer.fit(taxi_df_train_with_newFeatures) # Input data-frame is the cleaned one from above
	indexed = model.transform(taxi_df_train_with_newFeatures)
	encoder = OneHotEncoder(dropLast=False, inputCol="vendorIndex", outputCol="vendorVec")
	encoded1 = encoder.transform(indexed)
	
	# INDEX AND ENCODE RATE_CODE
	stringIndexer = StringIndexer(inputCol="rate_code", outputCol="rateIndex")
	model = stringIndexer.fit(encoded1)
	indexed = model.transform(encoded1)
	encoder = OneHotEncoder(dropLast=False, inputCol="rateIndex", outputCol="rateVec")
	encoded2 = encoder.transform(indexed)
	
	# INDEX AND ENCODE PAYMENT_TYPE
	stringIndexer = StringIndexer(inputCol="payment_type", outputCol="paymentIndex")
	model = stringIndexer.fit(encoded2)
	indexed = model.transform(encoded2)
	encoder = OneHotEncoder(dropLast=False, inputCol="paymentIndex", outputCol="paymentVec")
	encoded3 = encoder.transform(indexed)
	
	# INDEX AND TRAFFIC TIME BINS
	stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
	model = stringIndexer.fit(encoded3)
	indexed = model.transform(encoded3)
	encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
	encodedFinal = encoder.transform(indexed)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SAÍDA:**

Tempo necessário para executar a célula acima: 1,28 segundos

### Criar objetos de ponto rotulado para entrada em funções de ML

Esta seção contém código que mostra como indexar dados de texto categóricos como um tipo de dados de ponto rotulado e codificá-lo para que ele possa ser usado para treinar e testar a regressão logística de MLlib e outros modelos de classificação. Objetos de ponto rotulados são RDDs (Conjuntos de Dados Resilientes Distribuídos) formatados de uma maneira que é necessária como dados de entrada pela maioria dos algoritmos de ML no MLlib. Um [ponto rotulado](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) é um vetor local, denso ou esparso, associado a um rótulo/resposta.

Esta seção contém código que mostra como indexar dados de texto categóricos como um tipo de dados de [ponto rotulado](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) e codificá-lo para que ele possa ser usado para treinar e testar a regressão logística de MLlib e outros modelos de classificação. Objetos de ponto de rotulado são RDDs (Conjuntos de Dados Resilientes Distribuídos) que consistem em um rótulo (variável de destino/resposta) e um vetor de recurso. Esse formato é necessário como entrada para muitos algoritmos de ML no MLlib.

Este é o código para indexar e codificar recursos de texto para a classificação binária.

	# FUNCTIONS FOR BINARY CLASSIFICATION

	# LOAD LIBRARIES
	from pyspark.mllib.regression import LabeledPoint
	from numpy import array

	# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
	def parseRowIndexingBinary(line):
	    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex,
	                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
	                         line.trip_distance, line.fare_amount])
	    labPt = LabeledPoint(line.tipped, features)
	    return  labPt
	
	# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC RERESSION MODELS
	def parseRowOneHotBinary(line):
	    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
	                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
	                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
	                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
	    labPt = LabeledPoint(line.tipped, features)
	    return  labPt


Este é o código para codificar e indexar recursos de texto categórico para a análise de regressão linear.

	# FUNCTIONS FOR REGRESSION WITH TIP AMOUNT AS TARGET VARIABLE

	# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
	def parseRowIndexingRegression(line):
	    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
	                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
	                         line.trip_distance, line.fare_amount])

	    labPt = LabeledPoint(line.tip_amount, features)
	    return  labPt
	
	# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
	def parseRowOneHotRegression(line):
	    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
	                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
	                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
	                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
	    labPt = LabeledPoint(line.tip_amount, features)
	    return  labPt


### Criar uma subamostragem aleatória dos dados e dividi-la em conjuntos de treinamento e teste

Esse código cria uma amostragem aleatória dos dados (o valor de 25% é usado aqui). Embora não seja necessário para este exemplo, devido ao tamanho do conjunto de dados, demonstramos como é possível obter amostras aqui, para que você saiba como usá-lo para seu próprio problema, quando necessário. Quando as amostras são grandes, isso pode economizar um tempo significativo durante o treinamento de modelos. Em seguida, dividimos o exemplo em uma parte de treinamento (75% aqui) e uma parte de teste (25% aqui) para usar na classificação e na modelagem de regressão.


	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.sql.functions import rand

	# SPECIFY SAMPLING AND SPLITTING FRACTIONS
	samplingFraction = 0.25;
	trainingFraction = 0.75; testingFraction = (1-trainingFraction);
	seed = 1234;
	encodedFinalSampled = encodedFinal.sample(False, samplingFraction, seed=seed)
	
	# SPLIT SAMPLED DATA-FRAME INTO TRAIN/TEST
	# INCLUDE RAND COLUMN FOR CREATING CROSS-VALIDATION FOLDS (FOR USE LATER IN AN ADVANCED TOPIC)
	dfTmpRand = encodedFinalSampled.select("*", rand(0).alias("rand"));
	trainData, testData = dfTmpRand.randomSplit([trainingFraction, testingFraction], seed=seed);
	
	# FOR BINARY CLASSIFICATION TRAINING AND TESTING
	indexedTRAINbinary = trainData.map(parseRowIndexingBinary)
	indexedTESTbinary = testData.map(parseRowIndexingBinary)
	oneHotTRAINbinary = trainData.map(parseRowOneHotBinary)
	oneHotTESTbinary = testData.map(parseRowOneHotBinary)
	
	# FOR REGRESSION TRAINING AND TESTING
	indexedTRAINreg = trainData.map(parseRowIndexingRegression)
	indexedTESTreg = testData.map(parseRowIndexingRegression)
	oneHotTRAINreg = trainData.map(parseRowOneHotRegression)
	oneHotTESTreg = testData.map(parseRowOneHotRegression)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SAÍDA:**

Tempo necessário para executar a célula acima: 0,24 segundos


### Dimensionamento de recursos

O dimensionamento de recursos, também conhecido como normalização de dados, faz com que recursos com valores amplamente distribuídos não tenham peso excessivo na função objetiva. O código para o dimensionamento de recursos usa [StandardScaler](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.feature.StandardScaler) para dimensionar os recursos para variância de unidade. Ele é fornecido por MLlib para uso na regressão linear com SGD (Stochastic Gradient Descent), um algoritmo popular de treinamento de uma grande variedade de outros modelos de aprendizado de máquina, como regressões regularizadas ou SVM (máquinas de vetor de suporte).

>[AZURE.NOTE] Observamos que o algoritmo LinearRegressionWithSGD é sensível ao dimensionamento de recursos.

Aqui está o código para escalar as variáveis para uso com o algoritmo SGD linear regularizado.

	# FEATURE SCALING

	# RECORD START TIME
	timestart = datetime.datetime.now()

	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.regression import LabeledPoint
	from pyspark.mllib.linalg import Vectors
	from pyspark.mllib.feature import StandardScaler, StandardScalerModel
	from pyspark.mllib.util import MLUtils
	
	# SCALE VARIABLES FOR REGULARIZED LINEAR SGD ALGORITHM
	label = oneHotTRAINreg.map(lambda x: x.label)
	features = oneHotTRAINreg.map(lambda x: x.features)
	scaler = StandardScaler(withMean=False, withStd=True).fit(features)
	dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
	oneHotTRAINregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))
	
	label = oneHotTESTreg.map(lambda x: x.label)
	features = oneHotTESTreg.map(lambda x: x.features)
	scaler = StandardScaler(withMean=False, withStd=True).fit(features)
	dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
	oneHotTESTregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SAÍDA:**

Tempo necessário para executar a célula acima: 13,17 segundos


### Armazenar objetos em cache na memória

O tempo necessário para treinamento e teste dos algoritmos de ML pode ser reduzido armazenando-se em cache os objetos de quadro de dados de entrada usados para classificação, regressão e recursos dimensionados.

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# FOR BINARY CLASSIFICATION TRAINING AND TESTING
	indexedTRAINbinary.cache()
	indexedTESTbinary.cache()
	oneHotTRAINbinary.cache()
	oneHotTESTbinary.cache()
	
	# FOR REGRESSION TRAINING AND TESTING
	indexedTRAINreg.cache()
	indexedTESTreg.cache()
	oneHotTRAINreg.cache()
	oneHotTESTreg.cache()
	
	# SCALED FEATURES
	oneHotTRAINregScaled.cache()
	oneHotTESTregScaled.cache()
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SAÍDA:**

Tempo necessário para executar a célula acima: 0,15 segundos


## Prever se uma gorjeta é paga ou não com modelos de classificação binária

Esta seção mostra como usar três modelos para a tarefa de classificação binária de prever se uma gorjeta é paga ou não por uma corrida de táxi. Os modelos apresentados são:

- Regressão logística regularizada 
- Modelo de floresta aleatória
- Árvores de Ampliação de Gradiente

Cada seção de código de compilação de modelo será dividida em etapas:

1. Dados de **treinamento de modelo** com um conjunto de parâmetros
2. **Avaliação de modelo** em um conjunto de dados de teste com métricas
3. **Salvamento do modelo** no blob para consumo futuro

### Classificação usando regressão logística

O código nesta seção mostra como treinar, avaliar e salvar um modelo de regressão logística com [LBFGS](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm) que prevê se uma gorjeta é paga ou não por uma corrida no conjunto de dados de corridas e tarifas de táxi de Nova York.

**Treinar o modelo de regressão logística usando a CV e a limpeza de hiperparâmetro**

	# LOGISTIC REGRESSION CLASSIFICATION WITH CV AND HYPERPARAMETER SWEEPING

	# GET ACCURACY FOR HYPERPARAMETERS BASED ON CROSS-VALIDATION IN TRAINING DATA-SET

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD LIBRARIES
	from pyspark.mllib.classification import LogisticRegressionWithLBFGS 
	from sklearn.metrics import roc_curve,auc
	from pyspark.mllib.evaluation import BinaryClassificationMetrics
	from pyspark.mllib.evaluation import MulticlassMetrics
	
	
	# CREATE MODEL WITH ONE SET OF PARAMETERS
	logitModel = LogisticRegressionWithLBFGS.train(oneHotTRAINbinary, iterations=20, initialWeights=None, 
	                                               regParam=0.01, regType='l2', intercept=True, corrections=10, 
	                                               tolerance=0.0001, validateData=True, numClasses=2)
	
	# PRINT COEFFICIENTS AND INTERCEPT OF THE MODEL
	# NOTE: There are 20 coefficient terms for the 10 features, 
	#       and the different categories for features: vendorVec (2), rateVec, paymentVec (6), TrafficTimeBinsVec (4)
	print("Coefficients: " + str(logitModel.weights))
	print("Intercept: " + str(logitModel.intercept))
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**SAÍDA:**

Coeficientes: [0,0082065285375, -0,0223675576104, -0,0183812028036, -3,48124578069e-05, -0,00247646947233, -0,00165897881503, 0,0675394837328, -0,111823113101, -0,324609912762, -0,204549780032, -1,36499216354, 0,591088507921, -0,664263411392, -1,00439726852, 3,46567827545, -3,51025855172, -0,0471341112232, -0,043521833294, 0,000243375810385, 0,054518719222]

Interceptação: -0,0111216486893

Tempo necessário para executar a célula acima: 14,43 segundos

**Avaliar o modelo de classificação binária com métricas padrão**

	#EVALUATE LOGISTIC REGRESSION MODEL WITH LBFGS

	# RECORD START TIME
	timestart = datetime.datetime.now()

	# PREDICT ON TEST DATA WITH MODEL
	predictionAndLabels = oneHotTESTbinary.map(lambda lp: (float(logitModel.predict(lp.features)), lp.label))
	
	# INSTANTIATE METRICS OBJECT
	metrics = BinaryClassificationMetrics(predictionAndLabels)

	# AREA UNDER PRECISION-RECALL CURVE
	print("Area under PR = %s" % metrics.areaUnderPR)

	# AREA UNDER ROC CURVE
	print("Area under ROC = %s" % metrics.areaUnderROC)
	metrics = MulticlassMetrics(predictionAndLabels)

	# OVERALL STATISTICS
	precision = metrics.precision()
	recall = metrics.recall()
	f1Score = metrics.fMeasure()
	print("Summary Stats")
	print("Precision = %s" % precision)
	print("Recall = %s" % recall)
	print("F1 Score = %s" % f1Score)


	## SAVE MODEL WITH DATE-STAMP
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp;
	dirfilename = modelDir + logisticregressionfilename;
	logitModel.save(sc, dirfilename);
	
	# OUTPUT PROBABILITIES AND REGISTER TEMP TABLE
	logitModel.clearThreshold(); # This clears threshold for classification (0.5) and outputs probabilities
	predictionAndLabelsDF = predictionAndLabels.toDF()
	predictionAndLabelsDF.registerTempTable("tmp_results");
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds";

**SAÍDA:**

Área sob PR = 0,985297691373

Área sob ROC = 0,983714670256

Estatísticas de resumo

Precisão = 0,984304060189

Recuperação = 0,984304060189

Pontuação F1 = 0,984304060189

Tempo necessário para executar a célula acima: 57,61 segundos

**Plote a curva ROC.**

*predictionAndLabelsDF* é registrado como uma tabela, *tmp\_results*, na célula anterior. *tmp\_results* pode ser usado para fazer consultas e gerar resultados no quadro de dados sqlResults para plotagem. Veja o código.


	# QUERY RESULTS                              
	%%sql -q -o sqlResults
	SELECT * from tmp_results


Este é o código para fazer previsões e plotar a curva ROC.

	# MAKE PREDICTIONS AND PLOT ROC-CURVE

	# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
	%%local
	%matplotlib inline
	from sklearn.metrics import roc_curve,auc

	# MAKE PREDICTIONS
	predictions_pddf = test_predictions.rename(columns={'_1': 'probability', '_2': 'label'})
	prob = predictions_pddf["probability"] 
	fpr, tpr, thresholds = roc_curve(predictions_pddf['label'], prob, pos_label=1);
	roc_auc = auc(fpr, tpr)

	# PLOT ROC CURVE
	plt.figure(figsize=(5,5))
	plt.plot(fpr, tpr, label='ROC curve (area = %0.2f)' % roc_auc)
	plt.plot([0, 1], [0, 1], 'k--')
	plt.xlim([0.0, 1.0])
	plt.ylim([0.0, 1.05])
	plt.xlabel('False Positive Rate')
	plt.ylabel('True Positive Rate')
	plt.title('ROC Curve')
	plt.legend(loc="lower right")
	plt.show()
	

**SAÍDA:**

![curve.png de ROC de regressão logística](./media/machine-learning-data-science-spark-data-exploration-modeling/logistic-regression-roc-curve.png)


### Classificação de floresta aleatória

O código nesta seção mostra como treinar, avaliar e salvar um modelo de floresta aleatória que prevê se uma gorjeta é paga ou não por uma corrida no conjunto de dados de corridas e tarifas de táxi de Nova York.
	
	#PREDICT WHETHER A TIP IS PAID OR NOT USING RANDOM FOREST

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import RandomForest, RandomForestModel
	from pyspark.mllib.util import MLUtils
	from pyspark.mllib.evaluation import BinaryClassificationMetrics
	from pyspark.mllib.evaluation import MulticlassMetrics
	
	# SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	
	# TRAIN RANDOMFOREST MODEL
	rfModel = RandomForest.trainClassifier(indexedTRAINbinary, numClasses=2, 
	                                       categoricalFeaturesInfo=categoricalFeaturesInfo,
	                                       numTrees=25, featureSubsetStrategy="auto",
	                                       impurity='gini', maxDepth=5, maxBins=32)
	## UN-COMMENT IF YOU WANT TO PRINT TREES
	#print('Learned classification forest model:')
	#print(rfModel.toDebugString())
	
	# PREDICT ON TEST DATA AND EVALUATE
	predictions = rfModel.predict(indexedTESTbinary.map(lambda x: x.features))
	predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)
	
	# AREA UNDER ROC CURVE
	metrics = BinaryClassificationMetrics(predictionAndLabels)
	print("Area under ROC = %s" % metrics.areaUnderROC)
	
	# PERSIST MODEL IN BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	rfclassificationfilename = "RandomForestClassification_" + datestamp;
	dirfilename = modelDir + rfclassificationfilename;
	
	rfModel.save(sc, dirfilename);
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SAÍDA:**

Área sob ROC = 0,985297691373

Tempo necessário para executar a célula acima: 31,09 segundos


### Classificação de árvores de ampliação de gradiente

O código nesta seção mostra como treinar, avaliar e salvar um modelo de árvores de ampliação de gradiente que prevê se uma gorjeta é paga ou não por uma corrida no conjunto de dados de corridas e tarifas de táxi de Nova York.

	#PREDICT WHETHER A TIP IS PAID OR NOT USING GRADIENT BOOSTING TREES

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel
	
	# SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	
	gbtModel = GradientBoostedTrees.trainClassifier(indexedTRAINbinary, categoricalFeaturesInfo=categoricalFeaturesInfo, numIterations=5)
	## UNCOMMENT IF YOU WANT TO PRINT TREE DETAILS
	#print('Learned classification GBT model:')
	#print(bgtModel.toDebugString())
	
	# PREDICT ON TEST DATA AND EVALUATE
	predictions = gbtModel.predict(indexedTESTbinary.map(lambda x: x.features))
	predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)
	
	# AREA UNDER ROC CURVE
	metrics = BinaryClassificationMetrics(predictionAndLabels)
	print("Area under ROC = %s" % metrics.areaUnderROC)
	
	# PERSIST MODEL IN A BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp;
	dirfilename = modelDir + btclassificationfilename;
	
	gbtModel.save(sc, dirfilename)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 


**SAÍDA:**

Área sob ROC = 0,985297691373

Tempo necessário para executar a célula acima: 19,76 segundos


## Prever valores de gorjetas para corridas de táxi com modelos de regressão

Esta seção mostra como usar três modelos para a tarefa de regressão de prever o valor da gorjeta paga por uma corrida de táxi com base em outros recursos de gorjeta. Os modelos apresentados são:

- Regressão linear regularizada
- Floresta aleatória
- Árvores de Ampliação de Gradiente

Esses modelos foram descritos na introdução. Cada seção de código de compilação de modelo será dividida em etapas:

1. Dados de **treinamento de modelo** com um conjunto de parâmetros
2. **Avaliação de modelo** em um conjunto de dados de teste com métricas
3. **Salvamento do modelo** no blob para consumo futuro

### Regressão linear com SGD 

O código nesta seção mostra como usar recursos dimensionados para treinar uma regressão linear que usa SGD (Stochastic Gradient Descent) para otimização e como pontuar, avaliar e salvar o modelo no WASB (Armazenamento de Blobs do Azure).

>[AZURE.TIP] Em nossa experiência, pode haver problemas com a convergência de modelos LinearRegressionWithSGD, e os parâmetros precisam ser alterados/otimizados cuidadosamente para a obtenção de um modelo válido. O dimensionamento de variáveis ajuda significativamente com a convergência.


	#PREDICT TIP AMOUNTS USING LINEAR REGRESSION WITH SGD

	# RECORD START TIME
	timestart = datetime.datetime.now()
	
	# LOAD LIBRARIES
	from pyspark.mllib.regression import LabeledPoint, LinearRegressionWithSGD, LinearRegressionModel
	from pyspark.mllib.evaluation import RegressionMetrics
	from scipy import stats
	
	# USE SCALED FEATURES TO TRAIN MODEL
	linearModel = LinearRegressionWithSGD.train(oneHotTRAINregScaled, iterations=100, step = 0.1, regType='l2', regParam=0.1, intercept = True)

	# PRINT COEFFICIENTS AND INTERCEPT OF THE MODEL
	# NOTE: There are 20 coefficient terms for the 10 features, 
	#       and the different categories for features: vendorVec (2), rateVec, paymentVec (6), TrafficTimeBinsVec (4)
	print("Coefficients: " + str(linearModel.weights))
	print("Intercept: " + str(linearModel.intercept))
	
	# SCORE ON SCALED TEST DATA-SET & EVALUATE
	predictionAndLabels = oneHotTESTregScaled.map(lambda lp: (float(linearModel.predict(lp.features)), lp.label))
	testMetrics = RegressionMetrics(predictionAndLabels)
	
	# PRINT TEST METRICS
	print("RMSE = %s" % testMetrics.rootMeanSquaredError)
	print("R-sqr = %s" % testMetrics.r2)
	
	# SAVE MODEL WITH DATE-STAMP IN THE DEFAULT BLOB FOR THE CLUSTER
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
	dirfilename = modelDir + linearregressionfilename;
	
	linearModel.save(sc, dirfilename)
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SAÍDA:**

Coeficientes: [0.00457675809917, -0.0226314167349, -0.0191910355236, 0.246793409578, 0.312047890459, 0.359634405999, 0.00928692253981, -0.000987181489428, -0.0888306617845, 0.0569376211553, 0.115519551711, 0.149250164995, -0.00990211159703, -0.00637410344522, 0.545083566179, -0.536756072402, 0.0105762393099, -0.0130117577055, 0.0129304737772, -0.00171065945959]

Interceptação: 0,853872718283

RMSE = 1,24190115863

R-sqr = 0,608017146081

Tempo necessário para executar a célula acima: 58,42 segundos


### Regressão de Floresta Aleatória

O código nesta seção mostra como treinar, avaliar e salvar uma regressão de floresta aleatória que prevê o valor da gorjeta para os dados de corridas de táxi de Nova York.


	#PREDICT TIP AMOUNTS USING RANDOM FOREST

	# RECORD START TIME
	timestart= datetime.datetime.now()

	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import RandomForest, RandomForestModel
	from pyspark.mllib.util import MLUtils
	from pyspark.mllib.evaluation import RegressionMetrics
	
	
	## TRAIN MODEL
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
	                                    numTrees=25, featureSubsetStrategy="auto",
	                                    impurity='variance', maxDepth=10, maxBins=32)
	## UN-COMMENT IF YOU WANT TO PRING TREES
	#print('Learned classification forest model:')
	#print(rfModel.toDebugString())
	
	## PREDICT AND EVALUATE ON TEST DATA-SET
	predictions = rfModel.predict(indexedTESTreg.map(lambda x: x.features))
	predictionAndLabels = oneHotTESTreg.map(lambda lp: lp.label).zip(predictions)

	# TEST METRICS
	testMetrics = RegressionMetrics(predictionAndLabels)
	print("RMSE = %s" % testMetrics.rootMeanSquaredError)
	print("R-sqr = %s" % testMetrics.r2)
	
	# SAVE MODEL IN BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	rfregressionfilename = "RandomForestRegression_" + datestamp;
	dirfilename = modelDir + rfregressionfilename;
	
	rfModel.save(sc, dirfilename);
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SAÍDA:**

RMSE = 0,891209218139

R-sqr = 0,759661334921

Tempo necessário para executar a célula acima: 49,21 segundos


### Regressão de árvores de ampliação de gradiente

O código nesta seção mostra como treinar, avaliar e salvar um modelo de árvores de ampliação de gradiente que prevê o valor da gorjeta para os dados de corridas de táxi de Nova York.

**Treinar e avaliar**

	#PREDICT TIP AMOUNTS USING GRADIENT BOOSTING TREES

	# RECORD START TIME
	timestart= datetime.datetime.now()
	
	# LOAD PYSPARK LIBRARIES
	from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel
	from pyspark.mllib.util import MLUtils
	
	## TRAIN MODEL
	categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
	gbtModel = GradientBoostedTrees.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo, 
	                                                numIterations=10, maxBins=32, maxDepth = 4, learningRate=0.1)
	
	## EVALUATE A TEST DATA-SET
	predictions = gbtModel.predict(indexedTESTreg.map(lambda x: x.features))
	predictionAndLabels = indexedTESTreg.map(lambda lp: lp.label).zip(predictions)

	# TEST METRICS
	testMetrics = RegressionMetrics(predictionAndLabels)
	print("RMSE = %s" % testMetrics.rootMeanSquaredError)
	print("R-sqr = %s" % testMetrics.r2)

	# SAVE MODEL IN BLOB
	datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
	btregressionfilename = "GradientBoostingTreeRegression_" + datestamp;
	dirfilename = modelDir + btregressionfilename;
	gbtModel.save(sc, dirfilename)
	
	# CONVER RESULTS TO DF AND REGISER TEMP TABLE
	test_predictions = sqlContext.createDataFrame(predictionAndLabels)
	test_predictions.registerTempTable("tmp_results");
	
	# PRINT ELAPSED TIME
	timeend = datetime.datetime.now()
	timedelta = round((timeend-timestart).total_seconds(), 2) 
	print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 

**SAÍDA:**

RMSE = 0,908473148639

R-sqr = 0,753835096681

Tempo necessário para executar a célula acima: 34,52 segundos

**Plotar**

*tmp\_results* é registrado como uma tabela do Hive na célula anterior. Os resultados da tabela serão gerados no quadro de dados *sqlResults* para plotagem. Veja o código

	# PLOT SCATTER-PLOT BETWEEN ACTUAL AND PREDICTED TIP VALUES

	# SELECT RESULTS
	%%sql -q -o sqlResults
	SELECT * from tmp_results

Este é o código para plotar os dados usando o servidor do Jupyter.

	# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
	%%local
	%matplotlib inline
	import numpy as np

	# PLOT 
	ax = test_predictions_pddf.plot(kind='scatter', figsize = (6,6), x='_1', y='_2', color='blue', alpha = 0.25, label='Actual vs. predicted');
	fit = np.polyfit(test_predictions_pddf['_1'], test_predictions_pddf['_2'], deg=1)
	ax.set_title('Actual vs. Predicted Tip Amounts ($)')
	ax.set_xlabel("Actual")
	ax.set_ylabel("Predicted")
	ax.plot(test_predictions_pddf['_1'], fit[0] * test_predictions_pddf['_1'] + fit[1], color='magenta')
	plt.axis([-1, 20, -1, 20])
	plt.show(ax)
	

**SAÍDA:**

![Actual-vs-predicted-tip-amounts](./media/machine-learning-data-science-spark-data-exploration-modeling/actual-vs-predicted-tips.png)

	
## Limpar objetos da memória

Use `unpersist()` para excluir objetos armazenados em cache na memória.
		
	# REMOVE ORIGINAL DFs
	taxi_df_train_cleaned.unpersist()
	taxi_df_train_with_newFeatures.unpersist()
	
	# FOR BINARY CLASSIFICATION TRAINING AND TESTING
	indexedTRAINbinary.unpersist()
	indexedTESTbinary.unpersist()
	oneHotTRAINbinary.unpersist()
	oneHotTESTbinary.unpersist()
	
	# FOR REGRESSION TRAINING AND TESTING
	indexedTRAINreg.unpersist()
	indexedTESTreg.unpersist()
	oneHotTRAINreg.unpersist()
	oneHotTESTreg.unpersist()
	
	# SCALED FEATURES
	oneHotTRAINregScaled.unpersist()
	oneHotTESTregScaled.unpersist()


## Registre os locais de armazenamento dos modelos para consumo e pontuação

Para o consumo e a pontuação der um conjunto de dados independente descrito no tópico [Pontuar e avaliar modelos de aprendizado de máquina criados com Spark](machine-learning-data-science-spark-model-consumption.md), você precisará copiar e colar os nomes de arquivos que contêm os modelos salvos criados aqui para o bloco de anotações do Jupyter de Consumo. Aqui está o código para imprimir os caminhos para os arquivos de modelo necessários.

	# MODEL FILE LOCATIONS FOR CONSUMPTION
	print "logisticRegFileLoc = modelDir + "" + logisticregressionfilename + """;
	print "linearRegFileLoc = modelDir + "" + linearregressionfilename + """;
	print "randomForestClassificationFileLoc = modelDir + "" + rfclassificationfilename + """;
	print "randomForestRegFileLoc = modelDir + "" + rfregressionfilename + """;
	print "BoostedTreeClassificationFileLoc = modelDir + "" + btclassificationfilename + """;
	print "BoostedTreeRegressionFileLoc = modelDir + "" + btregressionfilename + """;


**SAÍDA**

logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS\_2016-05-0317\_03\_23.516568"

linearRegFileLoc = modelDir + "LinearRegressionWithSGD\_2016-05-0317\_05\_21.577773"

randomForestClassificationFileLoc = modelDir + "RandomForestClassification\_2016-05-0317\_04\_11.950206"

randomForestRegFileLoc = modelDir + "RandomForestRegression\_2016-05-0317\_06\_08.723736"

BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification\_2016-05-0317\_04\_36.346583"

BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression\_2016-05-0317\_06\_51.737282"


## O que vem a seguir?

Agora que criou modelos de regressão e classificação com o Spark MlLib, você está pronto para aprender a classificar e avaliar os modelos. O bloco de anotações de exploração e modelagem de dados avançadas se aprofunda na inclusão da validação cruzada, limpeza de hiperparâmetro e avaliação de modelo.

**Consumo de modelos:** para aprender a classificar e avaliar os modelos de classificação e regressão criados neste tópico, confira [Pontuar e avaliar modelos de aprendizado de máquina criados com Spark](machine-learning-data-science-spark-model-consumption.md).

**Validação cruzada e limpeza de hiperparâmetro**: consulte [Advanced data exploration and modeling with Spark (Modelagem e exploração de dados avançadas com Spark)](machine-learning-data-science-spark-advanced-data-exploration-modeling.md) sobre como os modelos podem ser treinados usando a validação cruzada e a limpeza de hiperparâmetro

<!---HONumber=AcomDC_0622_2016-->
<properties linkid="manage-services-hdinsight-get-started-hdinsight" urlDisplayName="Get Started" pageTitle="Get started using Hadoop 2.2 clusters with HDInsight | Azure" metaKeywords="" description="Get started using Hadoop 2.2 clusters with HDInsight, a big data solution. Learn how to provision clusters, run MapReduce jobs, and output data to Excel for analysis." metaCanonical="" services="hdinsight" documentationCenter="" title="Get started using Azure HDInsight" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />




# Get started using Hadoop 2.2 clusters with HDInsight 

HDInsight makes [Apache Hadoop][apache-hadoop] available as a service in the cloud. It makes the MapReduce software framework available in a simpler, more scalable, and cost efficient Azure environment. HDInsight also provides a cost efficient approach to the managing and storing of data using Azure Blob storage. 

In this tutorial, you will provision an HDInsight cluster using the Azure Management Portal, submit a Hadoop MapReduce job using PowerShell, and then import the MapReduce job output data into Excel for examination.

> [WACN.NOTE] This tutorial covers using Hadoop 2.2 clusters on HDInsight. For the tutorial using Hadoop 1.2 clusters on HDInsight, see [Get started using Azure HDInsight][hdinsight-get-started].

> [WACN.NOTE]	The *asv://* syntax is not supported in HDInsight clusters version 3.0 and will not be supported in later versions. The *wasb://* syntax should be used instead.

In conjunction with the general availability of Azure HDInsight, Microsoft has also released HDInsight Emulator for Azure, formerly known as Microsoft HDInsight Developer Preview. This product targets developer scenarios and as such only supports single-node deployments. For using HDInsight Emulator, see [Get Started with the HDInsight Emulator][hdinsight-emulator].

**Prerequisites:**

Before you begin this tutorial, you must have the following:


- An Azure subscription. For more information about obtaining a subscription, see [Purchase Options][azure-purchase-options], [Free Trial][azure-free-trial].
- A computer that is running Windows 8, Windows 7, Windows Server 2012, or Windows Server 2008 R2. This computer will be used to submit MapReduce jobs.
- Office 2013 Professional Plus, Office 365 Pro Plus, Excel 2013 Standalone, or Office 2010 Professional Plus.

**Estimated time to complete:** 30 minutes

##In this tutorial

* [Set up local environment for running PowerShell](#setup)
* [Provision an HDInsight cluster](#provision)
* [Run a WordCount MapReduce program](#sample)
* [Connect to Microsoft business intelligence tools](#powerquery)
* [Next steps](#nextsteps)



##<a id="setup"></a> Set up local environment for running PowerShell

There are several ways to submit MapReduce jobs to HDInsight. In this tutorial, you will use Azure PowerShell. To install Azure PowerShell, run the [Microsoft Web Platform Installer][powershell-download]. Click **Run** when prompted, click **Install**, and then follow the instructions. For more information, see [Install and configure Azure PowerShell][powershell-install-configure].

The PowerShell cmdlets require your subscription information so that it can be used to manage your services.

**To connect to your subscription using Azure AD**

1. Open the Azure PowerShell console, as instructed in [How to: Install Azure PowerShell][powershell-open].
2. Run the following command:

		Add-AzureAccount

3. In the window, type the email address and password associated with your account. Azure authenticates and saves the credential information, and then closes the window.

The other method to connect to  your subscription is using the certificate method. For instructions, see [Install and configure Azure PowerShell][powershell-install-configure].
	
##<a name="provision"></a>Provision an HDInsight cluster

The HDInsight provision process requires an Azure Storage account to be used as the default file system. The storage account must be located in the same data center as the HDInsight compute resources. Currently, you can only provision HDInsight clusters in the following data centers:

- China East
- China North

You must choose one of the five data centers for your Azure Storage account.

**To create an Azure Storage account**

1. Sign in to the [Azure Management Portal][azure-management-portal].
2. Click **NEW** on the lower left corner, point to **DATA SERVICES**, point to **STORAGE**, and then click **QUICK CREATE**.

	![HDI.StorageAccount.QuickCreate][image-hdi-storageaccount-quickcreate]

3. Enter **URL**, **LOCATION** and **REPLICATION**, and then click **CREATE STORAGE ACCOUNT**. Affinity groups are not supported. You will see the new storage account in the storage list. 
4. Wait until the **STATUS** of the new storage account is changed to **Online**.
5. Click the new storage account from the list to select it.
6. Click **MANAGE ACCESS KEYS** from the bottom of the page.
7. Make a note of the **STORAGE ACCOUNT NAME** and the **PRIMARY ACCESS KEY**.  You will need them later in the tutorial.


For the detailed instructions, see
[How to Create a Storage Account][azure-create-storageaccount] and [Use Azure Blob Storage with HDInsight][hdinsight-storage].


















Provision HDInsight 3.0 clusters is currently only supported using the custom create option.

**To provision an HDInsight cluster** 

1. Sign in to the [Azure Management Portal][azure-management-portal]. 

2. Click **HDINSIGHT** on the left to list the HDInsight clusters under your account. In the following screenshot, there is no existing HDInsight cluster.

	![HDI.ClusterStatus][image-hdi-clusterstatus]

3. Click **NEW** on the lower left side, click **DATA SERVICES**, click **HDINSIGHT**, and then click **CUSTOM CREATE**.

	![HDI.CustomCreateCluster][image-hdi-customcreatecluster]

4. From the Cluster Details tab, enter or select the following values:

	<table border="1">
	<tr><th>Name</th><th>Value</th></tr>
	<tr><td><strong>Cluster Name</strong></td><td>Name of the cluster.</td></tr>
	<tr><td><strong>Data Nodes</strong></td><td>Number of data nodes you want to deploy. For testing purposes, create a single node cluster. <br />The cluster size limit varies for Azure subscriptions. Contact Azure billing support to increase the limit.</td></tr>
	<tr><td><strong>HDInsight Version</strong></td><td>Choose <strong>3.0</strong> to create a Hadoop 2.2 cluster on HDInsight.</td></tr>
	<tr><td><strong>Region</strong></td><td>Choose the same region as the storage account you created in the last procedure. HDInsight requires the storage account located in the same region. Later in the configuration, you can only choose a storage account that is in the same region as you specified here.
	</td></tr>
	</table>

4. Click the right arrow in the bottom right corner to configure cluster user. 
4. From the Configure Cluster user tab, enter **User Name** and **Password** for the HDInsight cluster user account. In addition to this account, you can create a RDP user account after the cluster is provisioned, so you can remote desktop into the cluster. For instructions, see [Administer HDInsight using Management portal][hdinsight-admin-portal]
4. Click the right arrow in the bottom right corner to configure the storage account. 
5. From the Storage Account tag, enter or select the following values:

	<table border="1">
	<tr><th>Name</th><th>Value</th></tr>
	<tr><td>STORAGE ACCOUNT</td><td>Select <strong>Use Existing Storage</strong>. You also have the option to have the management portal to create a new storage account if you don't have one created.</td></tr>
	<tr><td>ACCOUNT NAME</td><td>Specify the storage account you created in the last procedure of this tutorial. Note only the storage accounts in the same region are displayed in the list box.</td></tr>
	<tr><td>DEFAULT CONTAINER</td><td>Select <strong>Create defatul container</strong>. When this options is chosen, the default container name has the same name as the cluster name.</td></tr>
	<tr><td>ADDITIONAL STORATGE ACCOUNT</td><td>Select <strong>0</strong>. You have the option to connect the cluster to up to 7 additional storage acounts.</td></tr>
	</table>

5. Click the check icon in the bottom right corner to create the cluster. When the provision process completes, the  status column will show **Running**.








##<a name="sample"></a>Run a WordCount MapReduce job

Now you have an HDInsight cluster provisioned. The next step is to run a MapReduce job to count words in a text file. 

Running a MapReduce job requires the following elements:

* A MapReduce program. In this tutorial, you will use the WordCount sample that comes with the HDInsight cluster distribution so you don't need to write your own. It is located on */example/jars/hadoop-mapreduce-examples.jar*. For instructions on writing your own MapReduce job, see [Develop Java MapReduce programs for HDInsight][hdinsight-develop-MapReduce].

* An input file. You will use */example/data/gutenberg/davinci.txt* as the input file. For information on upload files, see [Upload Data to HDInsight][hdinsight-upload-data].
* An output file folder. You will use */example/data/WordCountOutput* as the output file folder. The system will create the folder if it doesn't exist.

The URI scheme for accessing files in Blob storage is:

	wasb[s]://<containername>@<storageaccountname>.blob.core.chinacloudapi.cn/<path>

> [WACN.NOTE] By default, the Blob container used for the default file system has the same name as the HDInsight cluster.

The URI scheme provides both unencrypted access with the *wasb:* prefix, and SSL encrypted access with wasbs. We recommend using wasbs wherever possible, even when accessing data that lives inside the same Azure data center.

Because HDInsight uses a Blob Storage container as the default file system, you can refer to files and directories inside the default file system using relative or absolute paths.

For example, to access the hadoop-mapreduce-examples.jar, you can use one of the following options:

	● wasb://<containername>@<storageaccountname>.blob.core.chinacloudapi.cn/example/jars/hadoop-mapreduce-examples.jar
	● wasb:///example/jars/hadoop-mapreduce-examples.jar
	● /example/jars/hadoop-mapreduce-examples.jar
				
The use of the *wasb://* prefix in the paths of these files. This is needed to indicate Azure Blob Storage is being used for input and output files. The output directory assumes a default path relative to the *wasb:///user/&lt;username&gt;* folder. 

For more information, see [Use Azure Blob Storage with HDInsight][hdinsight-storage].





















**To run the WordCount sample**

1. Open **Azure PowerShell**. For instructions of opening Azure PowerShell console window, see [Install and configure Azure PowerShell][powershell-install-configure].

3. Run the following commands to set the variables.  
		
		$subscriptionName = "<SubscriptionName>" 
		$clusterName = "<HDInsightClusterName>"        
		
5. Run the following commands to create a MapReduce job definition:

		# Define the MapReduce job
		$wordCountJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-mapreduce-examples.jar" -ClassName "wordcount" -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data/WordCountOutput"

	The hadoop-mapreduce-examples.jar file comes with the HDInsight cluster distribution. There are two arguments for the MapReduce job. The first one is the source file name, and the second is the output file path. The source file comes with the HDInsight cluster distribution, and the output file path will be created at the run-time.

6. Run the following command to submit the MapReduce job:

		# Submit the job
		Select-AzureSubscription $subscriptionName
		$wordCountJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $wordCountJobDefinition 
		
	In addition to the MapReduce job definition, you must also provide the HDInsight cluster name where you want to run the MapReduce job. 

	The *Start-AzureHDInsightJob* is an asynchroized call.  To check the completion of the job, use the *Wait-AzureHDInsightJob* cmdlet.

6. Run the following command to check the completion of the MapReduce job:

		Wait-AzureHDInsightJob -Job $wordCountJob -WaitTimeoutInSeconds 3600 
		
8. Run the following command to check any errors with running the MapReduce job:	
	
		# Get the job output
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $wordCountJob.JobId -StandardError
		
	The following screenshot shows the output of a successful run. Otherwise, you will see some error messages.

	![HDI.GettingStarted.RunMRJob][image-hdi-gettingstarted-runmrjob]
































**To retrieve the results of the MapReduce job**

1. Open **Azure PowerShell**.
2. Run the following commands to create a C:\Tutorials folder, and change directory to the folder:

		mkdir \Tutorials
		cd \Tutorials
	
	The default Azure Powershell directory is *C:\Windows\System32\WindowsPowerShell\v1.0*. By default, you don't have the write permission on this folder. You must change directory to a folder where you have write permission.
	
2. Set the three variables in the following commands, and then run them:

		$subscriptionName = "<SubscriptionName>"       
		$storageAccountName = "<StorageAccountName>"   
		$containerName = "<ContainerName>"			   

	The Azure Storage account is the one you created earlier in the tutorial. The storage account is used to host the Blob container that is used as the default HDInsight cluster file system.  The Blob storage container name usually share the same name as the HDInsight cluster unless you specify a different name when you provision the cluster.

3. Run the following commands to create an Azure storage context object:
		
		# Create the storage account context object
		Select-AzureSubscription $subscriptionName
		$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
		$storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

	The *Select-AzureSubscription* is used to set the current subscription in case you have multiple subscriptions, and the default subscription is not the one to use. 

4. Run the following command to download the MapReduce job output from the Blob container to the workstation:

		# Download the job output to the workstation
		Get-AzureStorageBlobContent -Container $ContainerName -Blob example/data/WordCountOutput/part-r-00000 -Context $storageContext -Force

	The *example/data/WordCountOutput* folder is the output folder specified when you run the MapReduce job. *part-r-00000* is the default file name for MapReduce job output.  The file will be download to the same folder structure on the local folder. For example, in the following screenshot, the current folder is the C root folder. The file will be downloaded to the *C:\example\data\WordCountOutput&#92;* folder.

5. Run the following command to print the MapReduce job output file:

		cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"

	![HDI.GettingStarted.MRJobOutput][image-hdi-gettingstarted-mrjoboutput]

	The MapReduce job produces a file named *part-r-00000* with the words and the counts.  The script uses the findstr command to list all of the words that contains *"there"*.


> [WACN.NOTE] If you open <i>./example/data/WordCountOutput/part-r-00000</i>, a multi-line output from a MapReduce job, in Notepad, you will notice the line breaks are not renter correctly. This is expected.


	
##<a name="powerquery"></a>Connect to Microsoft business intelligence tools 

The Power Query add-in for Excel can be used to export output from HDInsight into Excel where Microsoft Business Intelligence (BI) tools can be used to further process or display the results. 

You must have Excel 2010 or 2013 installed to complete this part of the tutorial. Here we will import the default Hive table that ships in HDInsight.

**To download Microsoft PowerQuery for Excel**

- Download Microsoft Power Query for Excel from the [Microsoft Download Center](http://www.microsoft.com/zh-cn/download/details.aspx?id=39379) and install it.

**To import HDInsight data**

1. Open Excel, and create a new blank workbook.
3. Click the **Power Query** menu, click **From Other Sources**, and then click **From Azure HDInsight**.

	![HDI.GettingStarted.PowerQuery.ImportData][image-hdi-gettingstarted-powerquery-importdata]

3. Enter the **Account Name** of the Azure Blob Storage Account associated with your cluster, and then click **OK**. This is the storage account you created earlier in the tutorial.
4. Enter the **Account Key** for the Azure Blob Storage Account, and then click **Save**. 
5. In the Navigator pane on the right, double-click the Blob storage container name. By default the container name is the same name as the cluster name. 

6. Locate **part-r-00000** in the **Name** column (the path is *.../example/data/WordCountOutput*), and then click **Binary** on the left of **part-r-00000**.

	![HDI.GettingStarted.PowerQuery.ImportData2][image-hdi-gettingstarted-powerquery-importdata2]

8. Right-click **Column1.1**, and then select **Rename**.
9. Change the name to **Word**.
10. Repeat the process to rename **Column1.2** to **Count**.

	![HDI.GettingStarted.PowerQuery.ImportData3][image-hdi-gettingstarted-powerquery-importdata3]

9. Click **Apply & Close** in the upper left corner. The query then imports the word counting MapReduce job output into Excel.

##<a name="nextsteps"></a>Next steps
In this tutorial, you have learned how to provision a cluster with HDInsight, run a MapReduce job on it, and import the results into Excel where they can be further processed and graphically displayed using BI tools. To learn more, see the following articles:

- [Get started with HDInsight][hdinsight-get-started]
- [Get started with the HDInsight Emulator][hdinsight-emulator]
- [Use Azure Blob storage with HDInsight][hdinsight-storage]
- [Administer HDInsight using PowerShell][hdinsight-admin-powershell]
- [Upload data to HDInsight][hdinsight-upload-data]
- [Use Hive with HDInsight][hdinsight-hive]
- [Use Pig with HDInsight][hdinsight-pig]
- [Use Oozie with HDInsight][hdinsight-oozie]
- [Develop C# Hadoop streaming MapReduce programs for HDInsight][hdinsight-develop-streaming]
- [Develop Java MapReduce programs for HDInsight][hdinsight-develop-mapreduce]


[hdinsight-get-started]: /en-us/documentation/articles/hdinsight-get-started/
[hdinsight-get-started-3.0]: /en-us/documentation/articles/hdinsight-get-started-30/
[hdinsight-provision]: /en-us/documentation/articles/hdinsight-provision-clusters/
[hdinsight-admin-powershell]: /en-us/documentation/articles/hdinsight-administer-use-powershell/
[hdinsight-upload-data]: /en-us/documentation/articles/hdinsight-upload-data/
[hdinsight-mapreduce]: /en-us/documentation/articles/hdinsight-use-mapreduce
[hdinsight-hive]: /en-us/documentation/articles/hdinsight-use-hive/
[hdinsight-pig]: /en-us/documentation/articles/hdinsight-use-pig/
[hdinsight-oozie]: /en-us/documentation/articles/hdinsight-use-oozie/
[hdinsight-storage]: /en-us/documentation/articles/hdinsight-use-blob-storage/
[hdinsight-emulator]: /en-us/documentation/articles/hdinsight-get-started-emulator/
[hdinsight-develop-streaming]: /en-us/documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/
[hdinsight-develop-mapreduce]: /en-us/documentation/articles/hdinsight-develop-deploy-java-mapreduce/
[hdinsight-admin-portal]: /en-us/documentation/articles/hdinsight-administer-use-management-portal/
[hdinsight-develop-streaming]: /en-us/documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/

[azure-purchase-options]: http://www.windowsazure.cn/zh-cn/pricing/overview/

<!--
[azure-member-offers]: https://www.windowsazure.com/en-us/pricing/member-offers/
-->

[azure-free-trial]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: /en-us/documentation/articles/storage-create-storage-account/ 

[apache-hadoop]: http://hadoop.apache.org/

[powershell-download]: http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409
[powershell-install-configure]: /en-us/documentation/articles/install-configure-powershell/
[powershell-open]: /en-us/documentation/articles/install-configure-powershell/#install

[image-hdi-storageaccount-quickcreate]: ./media/hdinsight-get-started-3.0/HDI.StorageAccount.QuickCreate.png
[image-hdi-clusterstatus]: ./media/hdinsight-get-started-3.0/HDI.ClusterStatus.png
[image-hdi-customcreatecluster]: ./media/hdinsight-get-started-3.0/HDI.CustomCreateCluster.png
[image-hdi-wordcountdiagram]: ./media/hdinsight-get-started-3.0/HDI.WordCountDiagram.gif
[image-hdi-gettingstarted-mrjoboutput]: ./media/hdinsight-get-started-3.0/HDI.GettingStarted.MRJobOutput.png
[image-hdi-gettingstarted-runmrjob]: ./media/hdinsight-get-started-3.0/HDI.GettingStarted.RunMRJob.png
[image-hdi-gettingstarted-powerquery-importdata]: ./media/hdinsight-get-started-3.0/HDI.GettingStarted.PowerQuery.ImportData.png
[image-hdi-gettingstarted-powerquery-importdata2]: ./media/hdinsight-get-started-3.0/HDI.GettingStarted.PowerQuery.ImportData2.png
[image-hdi-gettingstarted-powerquery-importdata3]: ./media/hdinsight-get-started-3.0/HDI.GettingStarted.PowerQuery.ImportData3.png
# databricksazuretogcp
Azure to GCP(Google Platform) copy of files using databricks

This approach i have tested with interactive cluster of Databricks and need to complete the work for the Job cluster. 

Before this step i created a service account in google and downloaded the json file. 
Uploaded the json file to Azure blob storage. 
created a notebook in azure databricks using the following command. 

		○ Command 1, mount the azure storage. 
		storage_account = "datalakekuma"
		container = "movie"
		
		# the second piece of information needed is the blob SAS token. This will be a Shared Access Signature token found in the azure portal
		blobKey = "SASKeyfromAzureforthestorageaccount"
		
		blobEndpoint = "wasbs://{1}@{0}.blob.core.windows.net/".format(storage_account, container)
		
		try:
		  dbutils.fs.mount(
		    source = blobEndpoint,
		    mount_point = "/mnt/movies",
		    extra_configs = {"fs.azure.sas.{1}.{0}.blob.core.windows.net".format(storage_account, container):blobKey})
		except:
		  print("Already mounted")
		  pass
		○ Command 2, for me the same location is where my blob are store which needs to be uploaded. I copy that to the local file on the spark node. 
		#copy the file from the blob storage to local file system of spark engine, when we run the python code spark only understand the local file system. Need to see how the parallel activity would work, my init file is not working and needs more time. 
		dbutils.fs.cp("/mnt/movies/input/PowerBIQuery.txt","file:/databricks/data/PowerBIQuery.txt")
		
		○ Next moved the json file which is required for the authentication to google to the databricks file system . 
		#My private key for the service account was on Azure. After mounting the file system, copied the files to dbfs, may be this step is not requried and we can directly copy to the file system. But this ensure that dbfs always have the files after restart too
		dbutils.fs.cp("dbfs:/mnt/movies/movies/gtest.json","dbfs:/databricks/gtest.json")
		
		○ Next move the json file from Databricks file system to the spark file system / worker nodes. 
		#moved the private key to the spark nodes to the local filesystem, the root didn't work for me so i had to use /databricks/data folder
		dbutils.fs.cp("dbfs:/databricks/gtest.json","file:/databricks/data/gtest.json")
		
		○ This is where we can move the files to the google cloud platform
		# The authentication module and storage module. Then upload code, i also tried to list the blob which is there as part of the comment. 
		
		!pip install google-api-python-client
		!pip install google-cloud-storage
		import google.auth
		from google.cloud import storage
		bucket_name = 'rajukumamovies'
		blobToUpload='PowerBIQueryMoved.txt'
		
		storage_client = storage.Client.from_service_account_json('/databricks/data/gtest.json')
		bucket = storage_client.bucket(bucket_name)
		blob = bucket.blob(blobToUpload)
		    
		blob.upload_from_filename('/databricks/data/PowerBIQuery.txt')

Manually running it does work, need to validate if i run it from a datafactory as interactive cluster does that works too. 

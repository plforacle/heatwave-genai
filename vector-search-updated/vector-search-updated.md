# Perform Retrieval Augmented Generation 

## Introduction

Using the in-database vector store and retrieval-augmented generation (RAG), you can load and query unstructured documents stored in Object Storage using natural language within the HeatWave ecosystem.

_Estimated Time:_ 30 minutes

### Objectives

In this lab, you will be guided through the following task:

- Download HeatWave files.
- Create a bucket and a folder.
- Upload files to the bucket folder.
- Create a Pre-Authenticated Request (PAR) for secure access.
- Set up a vector store.
- Perform a vector search.

### Prerequisites

- Must complete Lab 4.

## Task 1: Download HeatWave files

Download HeatWave files on which we will perform a vector search.

1. Download the following files and save it in your local computer.

    - https://www.oracle.com/a/ocom/docs/mysql-heatwave-technical-brief.pdf
    - https://www.oracle.com/a/ocom/docs/mysql/mysql-heatwave-on-aws-brief.pdf
    - https://www.oracle.com/a/ocom/docs/mysql/mysql-heatwave-ml-technical-brief.pdf
    - https://www.oracle.com/a/ocom/docs/mysql/mysql-heatwave-lakehouse-technical-brief.pdf

    ![Download files](./images/vector-search-folder.png "Download files")

## Task 2: Create a bucket and a folder

The Object Storage service provides reliable, secure, and scalable object storage. Object Storage uses buckets to organize your files. 

1. Open the **Navigation menu** and click **Storage**. Under **Object Storage & Archive Storage**, click **Buckets**.

    ![Click bucket](./images/1-click-bucket.png "Click bucket")

2. In the **heatwave-genai** compartment, click **Create Bucket**. 

    ![Create bucket](./images/2-create-bucket.png "Create bucket")

3. Enter the **Bucket Name**, and accept the defaults for the rest of the fields.

    ```bash
    <copy>bucket-vector-search</copy>
    ```

4. Click **Create**.

    ![Enter bucket details](./images/3-enter-bucket-details.png "Enter bucket details")

5. Once the bucket is created, click the name of the bucket to open the **Bucket Details** page. 

    ![Created bucket](./images/4-created-bucket.png "Created bucket")

6. Copy the bucket name and **Namespace**, and paste the it somewhere for future reference.

    ![Bucket namespace](./images/35-bucket-namespace.png "Bucket namespace")

7. Under **Objects**, click **More Actions**, and then click **Create New Folder**.

    ![Create new folder](./images/31-create-new-folder.png "Create new folder")

8. In the **Create New Folder** page, enter a **Name** of the folder, and note it for future reference.

    ```bash
    <copy>bucket-folder-heatwave</copy>
    ```

    ![Enter new folder details](./images/32-enter-folder-details.png "Enter new folder details")

9. Click **Create**.

## Task 3: Upload files to the bucket folder

1.  In the **Bucket Details** page, under **Objects**, click the bucket folder name.

    ![Click bucket folder](./images/33-click-bucket-folder.png "Click bucket folder")

2.  Click **Upload**.

    ![Click upload](./images/34-click-upload.png "Click upload")

3. Click **select files** to display a file selection dialog box.

4. Select the files you had downloaded earlier in Task 1, and click **Upload**.

    ![Upload files](./images/6-upload-files.png "Upload files")

5. When the file status shows **Finished**, click **Close** to return to the bucket.

    ![Upload files finished](./images/7-upload-files-finished.png "Upload files finished")


## Task 4: Create a Pre-Authenticated Request (PAR)

Pre-authenticated requests provide a way to let HeatWave access your bucket or objects without requiring dynamic groups or IAM policies. This is a simpler and more direct method for granting access.

1. In the **Bucket Details** page, under **Resources**, click **Pre-Authenticated Requests**, and then click **Create Pre-Authenticated Request**.

    ![Create Pre-Authenticated Request](./images/8-create-par.png "Create Pre-Authenticated Request")

2. In the **Create Pre-Authenticated Request** dialog box, do the following:
    
    **Name**: Enter a descriptive name for the PAR.
    
    ```bash
    <copy>heatwave-vector-search-par</copy>
    ```
    
    **Pre-Authenticated Request Target**: Select **Bucket** (this allows access to all files within the bucket and its folders).
    
    **Access Type**: Select **Permits object reads** (HeatWave only needs to read the files).
    
    **Enable Object Listing**: **Check this box** (this is required for HeatWave to list and access multiple files in the folder).
    
    **Expiration**: You can choose to increase the **Expiration** date of the Pre-Authenticated Request. Set an appropriate expiration date based on your usage requirements. The default is typically 7 days, but you may want to extend this for longer-term access.
    
    Click **Create Pre-Authenticated Request**. 

    ![Enter Pre-Authenticated Request details](./images/9-par-details.png "Enter Pre-Authenticated Request details")

3. **CRITICAL**: Copy the URL that appears in the **Pre-Authenticated Request Details** dialog box. 

    **Important Notes**:
    - This URL will **only be displayed once** and cannot be retrieved later.
    - Save this URL in a secure location for use in the next task.
    - The URL provides tokenized access to your bucket without requiring additional authentication.
    
    Paste the URL somewhere for future reference, and click **Close**.

    ![Copy Pre-Authenticated Request details](./images/10-copy-par.png "Copy Pre-Authenticated Request details")

    **Example PAR URL**:
    ```
    https://objectstorage.us-ashburn-1.oraclecloud.com/p/abc123def456.../n/your-namespace/b/bucket-vector-search/o/
    ```


## Task 5: Set up a vector store

1. Go to Visual Studio Code, and under **DATABASE CONNECTIONS**, click **Open New Database Connection** icon next to your HeatWave instance to connect to it. 

    ![Connect database](./images/vscode-connection.png "Connect database")

2. Create a new schema and select it.

    ```bash
    <copy>create database genai_db;</copy>
	<copy>use genai_db;</copy>
    ```

    ![Create database](./images/11-create-database.png "Create database")

2. Call the following procedure to create a schema that is used for task management.

    ```bash
    <copy>select mysql_task_management_ensure_schema();</copy>
    ```

    ![Call procedure](./images/12-call-procedure.png "Call procedure")

3. Click **Reload Database Information** to view the mysql\_task\_management schema.

    ![Reload Database Information](./images/13-reload-database-information.png "Reload Database Information")

4. Ingest the file from the Object Storage using the Pre-Authenticated Request URL, create vector embeddings, and load the vector embeddings into HeatWave:

    ```bash
    <copy>call sys.VECTOR_STORE_LOAD('<PAR-URL>', '{"table_name": "VectorStoreTableName"}');</copy>
    ```
    Replace the following:

    - **PAR-URL**: The complete Pre-Authenticated Request URL that you copied in Task 4, Step 3. Make sure to include the full URL.
    - **VectorStoreTableName**: The name you want for the vector store table.
   
    **Important**: If your files are in a folder within the bucket, you may need to append the folder path to the PAR URL:
    ```
    <PAR-URL>bucket-folder-heatwave/
    ```

    **Example**:

    ```bash
    <copy>call sys.VECTOR_STORE_LOAD('https://objectstorage.us-ashburn-1.oraclecloud.com/p/abc123def456.../n/axqzijr6ae84/b/bucket-vector-search/o/bucket-folder-heatwave/', '{"table_name": "livelab_embedding"}');</copy>
    ```

    ![Ingest files from Object Storage](./images/14-ingest-files.png "Ingest files from Object Storage")

    **Note**: The PAR URL provides secure, time-limited access to your Object Storage bucket without requiring dynamic groups or IAM policies.

5. Wait for a few minutes, and verify that embeddings are loaded in the vector embeddings table.

    ```bash
    <copy>select count(*) from <EmbeddingsTableName>;</copy>
    ```
    For example:
    ```bash
    <copy>select count(*) from livelab_embedding_pdf; </copy>
    ```
    It takes a couple of minutes to create vector embeddings. You should see a numerical value in the output, which means your embeddings are successfully loaded in the table.

    ![Vector embeddings](./images/15-check-count.png "Vector embeddings")

## Task 6: Perform retrieval augmented generation

HeatWave retrieves content from the vector store and provide that as context to the LLM. This process is called as retrieval-augmented generation or RAG. This helps the LLM to produce more relevant and accurate results for your queries.

1. Set the @options session variable to specify the table for retrieving the vector embeddings, and click **Enter**.

    ```bash
    <copy>set @options = JSON_OBJECT("vector_store", JSON_ARRAY("<DBName>.<EmbeddingsTableName>"));</copy>
    ```

    For example:

    ```bash
    <copy>set @options = JSON_OBJECT("vector_store", JSON_ARRAY("genai_db.livelab_embedding_pdf"));</copy>
    ```

2. Set the session @query variable to define your natural language query, and click **Enter**.

    ```bash
    <copy>set @query="<AddYourQuery>";</copy>
    ```
    
    Replace AddYourQuery with your natural language query.

    For example:

    ```bash
    <copy>set @query="What is HeatWave AutoML?";</copy>
    ```

3. Retrieve the augmented prompt, using the ML_RAG routine, and click **Execute the selection or full block on HeatWave and create a new block**.

    ```bash
    <copy>call sys.ML_RAG(@query,@output,@options);</copy>
    ```

    ![Set options](./images/17-set-options.png "Set options")

4. Print the output:

    ```bash
    <copy>select JSON_PRETTY(@output);</copy>
    ```
    
    Text-based content that is generated by the LLM in response to your query is printed as output. The output generated by RAG is comprised of two parts:

    - The text section contains the text-based content generated by the LLM as a response for your query.

    - The citations section shows the segments and documents it referred to as context.

    ![Vector search results](./images/18-vector-search-output.png "Vector search results")

## Important Notes on Pre-Authenticated Requests

- **Security**: PARs provide time-limited, tokenized access to your Object Storage. Anyone with the PAR URL can access the specified resources until the expiration date.
- **Expiration**: Make sure to set an appropriate expiration date. If the PAR expires, you'll need to create a new one and update your vector store configuration.
- **URL Management**: Store PAR URLs securely. They provide direct access to your data without additional authentication.
- **Revocation**: If needed, you can revoke a PAR before its expiration date from the OCI Console.
- **Scope**: The PAR created in this lab provides read-only access to the entire bucket, which is sufficient for HeatWave vector store operations.

## Learn More

- [HeatWave User Guide](https://dev.mysql.com/doc/heatwave/en/)

- [HeatWave on OCI User Guide](https://docs.oracle.com/en-us/iaas/mysql-database/index.html)

- [MySQL Documentation](https://dev.mysql.com/)

- [Oracle Cloud Infrastructure Pre-Authenticated Requests](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usingpreauthenticatedrequests.htm)


## Acknowledgements

- **Author** - Aijaz Fatima, Product Manager
- **Contributors** - Mandy Pang, Senior Principal Product Manager, Aijaz Fatima, Product Manager
- **Last Updated By/Date** - Updated for PAR usage, December 2024

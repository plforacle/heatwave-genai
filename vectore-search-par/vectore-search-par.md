# Perform Retrieval Augmented Generation (RAG) with PAR-Based Access

## Introduction
Using the in-database vector store and retrieval-augmented generation (RAG), you can load and query unstructured documents stored in Object Storage using natural language within the HeatWave ecosystem. This version uses a Pre-Authenticated Request (PAR) instead of dynamic groups or IAM policies.

Estimated Time: 30 minutes

## Objectives
- Download HeatWave PDF files.
- Create an Object Storage bucket and folder.
- Upload documents.
- Create a Pre-Authenticated Request (PAR) for read access.
- Set up a vector store in MySQL HeatWave.
- Perform a vector search and RAG query.

## Prerequisites
- Completion of Lab 4.
- A MySQL HeatWave DB System with HeatWave enabled.

## Task 1: Download HeatWave Files
Download the following PDF files to your local system:

- https://www.oracle.com/a/ocom/docs/mysql-heatwave-technical-brief.pdf
- https://www.oracle.com/a/ocom/docs/mysql/mysql-heatwave-on-aws-brief.pdf
- https://www.oracle.com/a/ocom/docs/mysql/mysql-heatwave-ml-technical-brief.pdf
- https://www.oracle.com/a/ocom/docs/mysql/mysql-heatwave-lakehouse-technical-brief.pdf

## Task 2: Create a Bucket and Folder
1. Open OCI Console → **Storage** → **Buckets**.
2. Click **Create Bucket**, name it for example:
   ```
   bucket-vector-search
   ```
3. Click the bucket, then create a folder:
   ```
   bucket-folder-heatwave
   ```

## Task 3: Upload Files
Upload the four PDF files into the folder created above.

## Task 4: Create a Pre-Authenticated Request (PAR)
1. Go to **Pre-Authenticated Requests** in the bucket.
2. Click **Create Pre-Authenticated Request**.
3. Choose:
   - Target: **Bucket** or Folder
   - Access: **Permit object reads**
   - Enable: **Object Listing**
4. Set an expiration date.
5. Copy the PAR URL shown, which looks like:
   ```
   https://objectstorage.<region>.oraclecloud.com/p/<PAR_TOKEN>/n/<namespace>/b/<bucket>/o/<folder>/
   ```

## Task 5: Set Up a Vector Store in MySQL HeatWave
1. Connect using VS Code or MySQL client.
2. Create a schema:
   ```sql
   CREATE DATABASE genai_db;
   USE genai_db;
   ```
3. Ensure task management schema:
   ```sql
   SELECT mysql_task_management_ensure_schema();
   ```
4. Load and embed documents using the PAR URL:
   ```sql
   CALL sys.VECTOR_STORE_LOAD(
     '<PAR_URL>',
     JSON_OBJECT("table_name", "livelab_embedding")
   );
   ```
5. Verify:
   ```sql
   SELECT COUNT(*) FROM livelab_embedding;
   ```

## Task 6: Perform RAG
1. Set RAG options:
   ```sql
   SET @options = JSON_OBJECT(
     "vector_store",
     JSON_ARRAY("genai_db.livelab_embedding")
   );
   ```
2. Set your question:
   ```sql
   SET @query = "What is HeatWave AutoML?";
   ```
3. Execute RAG:
   ```sql
   CALL sys.ML_RAG(@query, @output, @options);
   ```
4. View results:
   ```sql
   SELECT JSON_PRETTY(@output);
   ```

## Learn More
- HeatWave User Guide
- MySQL Documentation

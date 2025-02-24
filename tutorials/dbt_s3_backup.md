Step 1: Take backup of our DBT files into S3 

option 1 Go to S3 bucket and create your own folder. s3://dbt-training-airflow/DAG/

Do it through CLI 

Configure AWS credentials. 
aws configure 
AWS Access Key ID [None]:AKIAYS2NXCYD6I5DDKFZ
AWS Secret Access Key [None]: 
Default region name [None]: ap-south-1
Default output format [None]: json

Run this command to copy all files and folders into S3 
aws s3 cp . s3://dbt-training-airflow/DAG/vikash/ --recursive

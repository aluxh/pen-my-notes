# AWS SDK

The following is to demonstrate the infrastructure as code.

## Creating Redshift Cluster using the AWS python SDK

Few libraries you need are `pandas`, `boto3` (AWS SDK), `json` packages. And, the steps are as follows:

### Prepare the params and environment

1. Using an external `cfg` file to store all the params, and load into notebook using `configparser`.

- How the config file looks like:

```config
[AWS]
KEY=key
SECRET=secret

[DWH] 
DWH_CLUSTER_TYPE=multi-node
DWH_NUM_NODES=4
DWH_NODE_TYPE=dc2.large

DWH_IAM_ROLE_NAME=dwhRole
DWH_CLUSTER_IDENTIFIER=dwhCluster
DWH_DB=dwh
DWH_DB_USER=dwhuser
DWH_DB_PASSWORD=Passw0rd
DWH_PORT=5439
```

- How the python script looks like to load into the notebook:

```python
import configparser
config                  = configparser.ConfigParser()
config.read_file(open('dwh.cfg'))

KEY                     = config.get('AWS','KEY')
SECRET                  = config.get('AWS','SECRET')

DWH_CLUSTER_TYPE        = config.get("DWH","DWH_CLUSTER_TYPE")
DWH_NUM_NODES           = config.get("DWH","DWH_NUM_NODES")
DWH_NODE_TYPE           = config.get("DWH","DWH_NODE_TYPE")

DWH_CLUSTER_IDENTIFIER  = config.get("DWH","DWH_CLUSTER_IDENTIFIER")
DWH_DB                  = config.get("DWH","DWH_DB")
DWH_DB_USER             = config.get("DWH","DWH_DB_USER")
DWH_DB_PASSWORD         = config.get("DWH","DWH_DB_PASSWORD")
DWH_PORT                = config.get("DWH","DWH_PORT")

DWH_IAM_ROLE_NAME       = config.get("DWH", "DWH_IAM_ROLE_NAME")

(DWH_DB_USER, DWH_DB_PASSWORD, DWH_DB)

pd.DataFrame({"Param":
                 ["DWH_CLUSTER_TYPE", "DWH_NUM_NODES", "DWH_NODE_TYPE", "DWH_CLUSTER_IDENTIFIER", "DWH_DB", "DWH_DB_USER", "DWH_DB_PASSWORD", "DWH_PORT", "DWH_IAM_ROLE_NAME"],
                "Value":
                 [DWH_CLUSTER_TYPE, DWH_NUM_NODES, DWH_NODE_TYPE, DWH_CLUSTER_IDENTIFIER, DWH_DB, DWH_DB_USER, DWH_DB_PASSWORD, DWH_PORT, DWH_IAM_ROLE_NAME]
})
```

1. Create the resources and clients for EC2, S3, IAM and Redshift

   Using `boto3` library, create resources and clients in AWS.

```python
import boto3

ec2 = boto3.resource('ec2',
                region_name='us-west-2',
                aws_access_key_id=KEY,
                aws_secret_access_key=SECRET)

s3 = boto3.resource('s3',
                region_name='us-west-2',
                aws_access_key_id=KEY,
                aws_secret_access_key=SECRET)

iam = boto3.client('iam',
                region_name='us-west-2',
                aws_access_key_id=KEY,
                aws_secret_access_key=SECRET)

redshift = boto3.client('redshift',
                region_name='us-west-2',
                aws_access_key_id=KEY,
                aws_secret_access_key=SECRET)
```

1. Try printing out sample data sources from s3 using the following template:

```python
sampleDbBucket = s3.Bucket("bucket-name")
for obj in sampleDbBucket.objects.filter(Prefix="some prefix to filter data"):
        print (obj)
```

### Create Role and Attach Policy

Creating IAM role to allow AWS Redshift to call AWS services on your behalf.

```python
# Create Role
try:
    print('1.1 Creating a new IAM Role')
    dwhRole = iam.create_role(
        Path='/',
        RoleName=DWH_IAM_ROLE_NAME,
        Description="Allows Redshift clusters to call AWS services on your behalf.",
        AssumeRolePolicyDocument=json.dumps(
            {  'Statement': [{'Action': 'sts:AssumeRole',
               'Effect': 'Allow',
               'Principal': {'Service': 'redshift.amazonaws.com'}}],
               'Version': '2012-10-17'}
        )
    )

except Exception as e:
    print(e)

# Attach Policy
print('1.2 Attaching Policy')
iam.attach_role_policy(
    RoleName=DWH_IAM_ROLE_NAME,
    PolicyArn="arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
)['ResponseMetadata']['HTTPStatusCode']

# Get and print the IAM role ARN
print('1.3 Get the IAM role ARN')
roleArn = iam.get_role(RoleName=DWH_IAM_ROLE_NAME)['Role']['Arn']

print(roleArn)
```

### Create Redshift Cluster

Creating the redshift cluster using the standard template. 

```python
try:
    response = redshift.create_cluster(
        #HW
        ClusterType=DWH_CLUSTER_TYPE,
        NodeType=DWH_NODE_TYPE,
        NumberOfNodes=int(DWH_NUM_NODES),

        #Identifiers & Credentials
        DBName=DWH_DB,
        ClusterIdentifier=DWH_CLUSTER_IDENTIFIER,
        MasterUsername=DWH_DB_USER,
        MasterUserPassword=DWH_DB_PASSWORD,

        #Roles (for s3 access)
        IamRoles=[roleArn]  
)
except Exception as e:
    print(e)
```

Then, describe the cluster to examine the parameters and its status. Run this block several times until the cluster status becomes `Available`

```python
def prettyRedshiftProps(props):
    pd.set_option('display.max_colwidth', -1)
    keysToShow = ["ClusterIdentifier", "NodeType", "ClusterStatus", "MasterUsername", "DBName", "Endpoint", "NumberOfNodes", 'VpcId']
    x = [(k, v) for k,v in props.items() if k in keysToShow]
    return pd.DataFrame(data=x, columns=["Key", "Value"])

myClusterProps = redshift.describe_clusters(ClusterIdentifier=DWH_CLUSTER_IDENTIFIER)['Clusters'][0]
prettyRedshiftProps(myClusterProps)
```

Print out the cluster endpoint. **Only run this when the cluster is available**

```python
DWH_ENDPOINT = myClusterProps['Endpoint']['Address']
DWH_ROLE_ARN = myClusterProps['IamRoles'][0]['IamRoleArn']
print("DWH_ENDPOINT :: ", DWH_ENDPOINT)
print("DWH_ROLE_ARN :: ", roleArn)
```

### Open an incoming TCP port to access the cluster endpoint

```python
try:
    vpc = ec2.Vpc(id=myClusterProps['VpcId'])
    defaultSg = list(vpc.security_groups.all())[0]
    print(defaultSg)
    defaultSg.authorize_ingress(
        GroupName=defaultSg.group_name,
        CidrIp='0.0.0.0/0',
        IpProtocol='TCP',
        FromPort=int(DWH_PORT),
        ToPort=int(DWH_PORT)
    )
except Exception as e:
    print(e)
```

Test that you can access the cluster

```sql
%load_ext sql

conn_string="postgresql://{}:{}@{}:{}/{}".format(DWH_DB_USER, DWH_DB_PASSWORD, DWH_ENDPOINT, DWH_PORT,DWH_DB)
print(conn_string)
%sql $conn_string
```

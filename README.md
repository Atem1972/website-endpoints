- LETS CREATE CODEPIPLINE, STAGE1 CHOOSE BUILD CUSTOM PIPELINE THEN NEXT, STAGE2 PIPELINE NAME,CLICK QUEUED, NEW SERVICE ROLE THEN NEXT, SOURCE STAGE3, SOURCE PROVIDER GITHUB, CONNECT TO GITHUB, REPO NAME SELECT THE REPO WITH UR CODE, BRANCH MAIN, TICK BOX CODEPIPELINE DEFAULT THEN NEXT, SKIP BUILD STAGE, SKIP TEST, DEPLOY SLECT AMAZON S3, PUT REGION,PUT BUCKETNAME,TICK EXTRALFILE BEFOR DEPLOY AND TICK CONFIGURE AUTOMATIC , TO TEST IT CLON TE REPO PUT THIS 2 ENDPOINT AND PUSH BACK, TESTTER.GK, MYHOMEE.TK
- go to aws s3 and creat a bucket
- clone this code
- open the  file to see the content of the file
- copy the content of the file to the s3 bucket u created above ie RUN cp website-endpoints s3://<put the name of ur bucket here> --recursive
- create a role lambda then add police ESE, CLOUDWATCHFULLV2, S3
- LETS CREATE RULE EVENT TO TRIGER OUR LAMBDA, GO TO AMAZONEVENTBRIDGE, CLICK ON BUSES,CLICK ON RULES, CLICK CREATE RULE, ENTER THE NAME U WANT TO CALL UR RULE, CLICK ON THE BOX SCHEDULE, CLICK CONTINUS TO CREATE RULE, CLICK ON BOX A SCHEDULE THAT RUNS AT A REGULAR RATE --- , IN RATE EXPRESION SELECT UR MINS, CLICK NEXT, ON TARGET1 SELECT AWS SERVICE, ON SELECT TARGET SELECT LAMBDA FUNCTION, ON TARGET LOCATION SELECT TARGET IN THIS ACCOUNT, ON FUNCTION SELECT THE NAME OF UR LAMBDA FUNCTION U CREATED, CLICK ON SKIP TO REVIEW AND CREATE
- GO TO ESE SERVICE IN AWS AND CREATE UR EMAIL AND VERIFY IT IE, CLICK ON ESE, SELECT IDENTIFICATION,CLICK CREATE,SELECT EMAILL, PUT UR EMAIL, CLICK CREATE
- CREATE LAMDAT, TIMEOUT 5MINS
-   LAMBDA CODE,
-   
-  import json
import boto3
import socket

def lambda_handler(event, context):
    #calling the function with websites
    region="us-east-1"
    senderemail="enter sender email"
    receiveremail="enter receiver email"
    S3bucket="enter s3 bucketname"
    websiteFile="websites.txt"
    sites=get_bucket_data(S3bucket,websiteFile)
    if is_verified_email(senderemail,receiveremail):
        for site in sites:
            if is_running(f'{site}'):
                #send_plain_email(senderemail,receiveremail,"Website is up", f"{site} is running!",region)
                print(f"{site} is running!")
            else:
                send_plain_email(senderemail,receiveremail,"Website is down",f'There is a problem with {site}! please verify',region)
                print(f'There is a problem with {site}!')
    else:
        print("verification email sent")
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }


#function to get all the verified email.
def get_list_of_verified_emails():
    # Create SES client
    ses = boto3.client('ses')

    response = ses.list_verified_email_addresses()
    print(response)
    return(response['VerifiedEmailAddresses'])
    
#function to send verification email.    
def verify_email(email):
    # Create SES client
    ses = boto3.client('ses')
    
    response = ses.verify_email_identity(
      EmailAddress = email
    )

    print(response)
    
#check if email is verified and send verification.
def is_verified_email(sender_email_address,receiver_email_address):
    verified_emails= get_list_of_verified_emails()
    setemail=[sender_email_address,receiver_email_address]
    print(setemail)
    # verifies if sender and receiver are verified then sends the message.
    if set(setemail).issubset(set(verified_emails)):
        return True
        #send_plain_email(sender_email_address,receiver_email_address,subject,message)
    else:
        # sends sends verification emails to the unverified email.
        unverified_emails=list(set(setemail)- set(setemail).intersection(set(verified_emails)))
        print(unverified_emails)
        for email in unverified_emails:
            verify_email(email)
        return False
        
        

#function to test if website is running
def is_running(site):
    """This function attempts to connect to the given server using a socket.
        Returns: Whether or not it was able to connect to the server."""
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect((site, 80))
        return True
    except:
        return False
#function to get the  data from S3 bucket
def get_bucket_data(bucketName,bucketObject):
    # getting the S3 object
    s3 = boto3.client('s3')
    data = s3.get_object(Bucket=bucketName, Key=bucketObject)
    contents = data['Body'].read().decode("utf-8")
    websites = contents.splitlines()
    return websites
def send_plain_email(sender,receiver,subject,message,region):
    ses_client = boto3.client("ses", region_name=region)
    CHARSET = "UTF-8"
    try:
        response = ses_client.create_configuration_set(
            ConfigurationSet={
                'Name': 'my-config-set'
            }
        )   
    except Exception as e:
        print('Configuration set exists: ' + e.response['Error']['Message'])

    else:
        print(f'Configuration set creation error !!! Please check your configuration set')
    
    try:
        
        response = ses_client.send_email(
            Destination={
                "ToAddresses": [
                    receiver,
                ],
            },
            Message={
                "Body": {
                    "Text": {
                        "Charset": CHARSET,
                        "Data": message,
                    }
                },
                "Subject": {
                    "Charset": CHARSET,
                    "Data":subject,
                },
            },
            Source=sender,
            ConfigurationSetName='my-config-set',
        )   
    
    except Exception as e:
        print(e.response['Error']['Message'])
    else:
        print(f"Email sent! Message ID: {response['MessageId']}")

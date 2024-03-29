---
layout: post
title: "CIS AWS Foundations Benchmark" 
categories: AWS
date : 2022-03-25 07:13:50
---

First and foremost, Happy Greek National Day! 
(It just so happens today is the day)

This article emanates directly from a very real need I encountered when trying to manage 9 AWS accounts simulatniously. I wanted to make sure to set some alarms and alerts for each account, in order to somewhat automate the overall account security.

I reality, the origins of this need are nested in the AWS Security Hub and the `CIS AWS Foundations Benchmark standard` (but let's not make us look silly).

[Here](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-cis-controls.html) is a list of all the controls that the standard requires you to implement. It is easy to see how creating this manually would be a **major** pain in the back. 

In come,... CloudFormation. (yes, yes Vali, the day has come where you see me praise CloudFormation).

The template below will set the alarms and alerts required to satisfy the CIS AWS Foundations Benchmark. 

Disclamer: I did not create this! If I still had the source, I would definitely slap it here as it deserves credit!



```
---
AWSTemplateFormatVersion: '2010-09-09'
Description: Automate Section 3 - Monitoring of the CIS AWS Foundations Benchmark controls and send alarms to an email address.
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
    - Label:
        default: ALARM SNS TOPIC
      Parameters:
      - SetEmail
      - SetALARMTopicName
      - SetALARMTopicDisplayName
    - Label:
        default: CLOUDTRAIL SNS TOPIC
      Parameters:
      - SetTRAILTopicName
      - SetTRAILTopicDisplayName
    - Label:
        default: S3 & CLOUDTRAIL
      Parameters:
      - PreExistingS3BucketCloudTrail
      - CrossAccount01
      - CrossAccount02
      - CrossAccount03
      - CloudTrailLogGroupName
      - NameCloudTrailRole
    - Label:
        default: CLOUDWATCH
      Parameters:
      - CloudWatchLogsRetention
    - Label:
        default: NAME YOUR METRIC NAMESPACE 
      Parameters:  
      - NameYourNameSpace
    - Label:
        default: NAME YOUR ALARMS & METRIC FILTER NAMES
      Parameters:
      - 31UnauthAPICalls
      - 32MgmtConsoleNoMFA
      - 33UseOfRootAcct
      - 34IAMPolicyChanges
      - 35CloudTrailConfigChanges
      - 36ConsoleAuthFailures
      - 37DisableDeleteCMK
      - 38S3BucketPolicyChanges
      - 39AWSConfigChanges
      - 310SecGroupChanges
      - 311NACLChanges
      - 312NetworkGatewayChanges
      - 313RouteTableChanges
      - 314VPCChanges
    ParameterLabels:
      SetEmail:
        default: Alarm Email
      SetALARMTopicName:
        default: (OPTIONAL) Alarm Topic Name
      SetALARMTopicDisplayName:
        default: (SEMI-OPTIONAL) Alarm Topic Display Name
      SetTRAILTopicName:
        default: (OPTIONAL) Cloudtrail Topic Name
      SetTRAILTopicDisplayName:
        default: (SEMI-OPTIONAL) Cloudtrail Topic Display Name
      CloudWatchLogsRetention:
        default: CloudWatch Log Retention
      PreExistingS3BucketCloudTrail:
        default: CloudTrail S3 Bucket
      CrossAccount01:
        default: Cross-Account 01
      CrossAccount02:
        default: Cross-Account 02
      CrossAccount03:
        default: Cross-Account 03
      CloudTrailLogGroupName:
        default: (OPTIONAL) CloudTrail Log Group Name
      NameCloudTrailRole:
        default: CloudTrail Role Name
      NameYourNameSpace:
        default: Metric Name Space
      31UnauthAPICalls:
        default: Unauthorized API Calls
      32MgmtConsoleNoMFA:
        default: Management Console Sign-in w/o MFA
      33UseOfRootAcct:
        default: Usage of "root" Account
      34IAMPolicyChanges:
        default: IAM Policy Changes
      35CloudTrailConfigChanges:
        default: CloudTrail Configuration Changes
      37DisableDeleteCMK:
        default: Disabling / Deletion of CMKs
      38S3BucketPolicyChanges:
        default: S3 Bucket Policy Changes
      39AWSConfigChanges:
        default: AWS Config Changes
      310SecGroupChanges:
        default: Security Group Changes
      311NACLChanges:
        default: NACL Changes
      312NetworkGatewayChanges:
        default: Network Gateway Changes
      313RouteTableChanges:
        default: Route Table Changes
      314VPCChanges:
        default: VPC Changes

Parameters:
  
  SetEmail:
    Type: String
    Default: "example@example.com"
    Description: Triggered alarms will alert to this email address. Verification required and you can opt-out later. Make sure to validate otherwise you won't be able to delete the subscription and have to wait 3 days for automatic deletion.
    # REGEX Help:
    # We could get fancy with some serious REGEX action here but keep it simple.
    # This allows one or more characters (.+) in front and behind the @ symbol.
    AllowedPattern: ".+@.+"
    ConstraintDescription: You must enter one or more characters before and after the @ symbol. 
  
  SetALARMTopicName:
    Type: String
    Default: CIS-BENCHMARK-ALARMS
    Description: "Set a unique name for your CloudWatch Alarm SNS Topic. If you don't specify a name, a random name like <stack name>-AlarmNotificationTopic-9VWR9AVOB5QS will be automatically generated."
    MinLength: '0'
    MaxLength: '256'
    AllowedPattern: "[\\w_-]*"
  
  SetALARMTopicDisplayName:
    Type: String
    Default: cis-alarm
    Description: "REQUIRED for SMS. Can only use up to 10 characters from the AWS console."
    MinLength: '0'
    MaxLength: '10'
    AllowedPattern: "[\\w_-]*"
  
  SetTRAILTopicName:
    Type: String
    Default: CIS-BENCHMARK-CLOUDTRAIL
    Description: "Set a unique name for your CloudTrail SNS Topic. If you don't specify a name, a random name like <stack name>-TrailTopic-9VWR9AVOB5QS will be automatically generated."
    MinLength: '0'
    MaxLength: '256'
    AllowedPattern: "[\\w_-]*"
  
  SetTRAILTopicDisplayName:
    Type: String
    Default: cis-cldtrl
    Description: "REQUIRED for SMS. Can only use up to 10 characters from the AWS console."
    MinLength: '0'
    MaxLength: '10'
    AllowedPattern: "[\\w_-]*"
  
  CloudWatchLogsRetention:
    Description: The number of days log events are kept in CloudWatch Logs
    Type: Number
    Default: 90
    AllowedValues:
    - 1
    - 3
    - 5
    - 7
    - 14
    - 30
    - 60
    - 90
    - 120
    - 150
    - 180
    - 365
    - 400
    - 545
    - 731
    - 1827
    - 3653
  
  PreExistingS3BucketCloudTrail:
    #Description: "Enter an unique name for your EXISTING CloudTrail S3 bucket that only contains lowercase letters, numbers, periods (.), and dashes (-). If you don't specify a name, a random name like \"<stack name>-cloudtrails3bucket-2wpe5iffv606\" will be automatically generated. Note that this bucket's DeletionPolicy is set to Retain. (i.e. Stack deletion will not delete the bucket.)"
    Description: "Enter the name of your EXISTING root S3 bucket for S3 log collection or the export name of the bucket to send logs to. Note that the templates have to be in the same region."
    Type: String
    # Typically S3 bucket names must be between 3 and 63 characters but since we want to allow an empty string, set MinLength to '0'.
    # Ref: http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-s3-bucket-naming-requirements.html
    MinLength: '0'
    MaxLength: '63'
    # REGEX Help:
    # This allows one or more (+) lower case alphanumeric characters (a-z0-9), no spaces, and the characters '.' (period) and '-' (hyphen) OR an empty string: ^(|)$
    AllowedPattern: "^(|[a-z0-9.-]+)$"
    ConstraintDescription: The bucket name must contain only lowercase letters, numbers, periods (.), and dashes (-).
  
  CrossAccount01:
    Description: "Enter the account name from where you'd like to receive CloudTrail logs. Leave empty if none."
    Type: String
    AllowedPattern: "(|\\d{12})"
    ConstraintDescription: AWS accounts are 12 digits long. Enter 12 digits or leave blank.

  CrossAccount02:
    Description: "Enter the account name from where you'd like to receive CloudTrail logs. Leave empty if none."
    Type: String
    AllowedPattern: "(|\\d{12})"
    ConstraintDescription: AWS accounts are 12 digits long. Enter 12 digits or leave blank.

  CrossAccount03:
    Description: "Enter the account name from where you'd like to receive CloudTrail logs. Leave empty if none."
    Type: String
    AllowedPattern: "(|\\d{12})"
    ConstraintDescription: AWS accounts are 12 digits long. Enter 12 digits or leave blank.

  CloudTrailLogGroupName:
    Description: Enter a name for your CloudTrail LogGroup.
    Type: String
    Default: CloudTrail/DefaultLogGroup
    MinLength: '0'
    MaxLength: '512'
    AllowedPattern: "[\\w-/.]+"
    ConstraintDescription: Log group names can be between 1 and 512 characters long.
      Allowed characters include a-z, A-Z, 0-9, '_' (underscore), '-' (hyphen), '/'
      (forward slash), and '.' (period).
  
  NameCloudTrailRole:
    Description: "A unique name for a CloudTrail Role to be created that contains alphanumeric characters, no spaces, and the characters: = , . @ -. If you don't specify a name, a random name like \"<stack name>-TrailLogGroupRole-1OFN28AF4AUPV\" will be automatically generated."
    Type: String
    MinLength: '0'
    MaxLength: '64'
    # REGEX Help:
    # This allows one or more (+) alphanumeric characters (\w) with the symbols =,.@- OR an empty string: ^(|)$
    AllowedPattern: "^(|[|\\w=,.@-]+)$"
    ConstraintDescription: "The role name must contain only upper and lowercase alphanumeric characters with no spaces. These characters are alllowed: = , . @ -"
  
  NameYourNameSpace:
    Description: "A namespace is a container for CloudWatch metrics. Metrics in different namespaces are isolated from each other, so that metrics from different applications are not mistakenly aggregated into the same statistics. Possible characters are: alphanumeric characters (0-9A-Za-z), period (.), hyphen (-), underscore (_), forward slash (/), hash (#), and colon (:)."
    Type: String
    MaxLength: '256'
    Default: CIS-3-MONITORING
  
  31UnauthAPICalls:
    Description: 3.01 Ensure a log metric filter and alarm exist for unauthorized API calls 
    Type: String
    Default: CIS Benchmark v.1.1 - 3.01 Unauthorized API Calls
    MinLength: '1'
    MaxLength: '255'
    # REGEX Help:
    # This allows one or more (+) alphanumeric characters (\w) with the symbols -_/.
    # Must escape "\" and symbols with "\"
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  32MgmtConsoleNoMFA:
    Description: 3.02 Ensure a log metric filter and alarm exist for Management Console sign-in without MFA (Scored)
    Type: String
    Default: CIS Benchmark v.1.1 - 3.02 Management Console Sign-in Without MFA
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  33UseOfRootAcct:
    Description: 3.03 Ensure a log metric filter and alarm exist for usage of "root" account (Scored)
    Type: String
    # Be careful - can't use quotes in Metric Filter name so "root" becomes root.
    Default: CIS Benchmark v.1.1 - 3.03 Usage of root Account
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  34IAMPolicyChanges:
    Description: 3.04 Ensure a log metric filter and alarm exist for IAM policy changes (Scored)
    Type: String
    Default: CIS Benchmark v.1.1 - 3.04 IAM Policy Changes
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  35CloudTrailConfigChanges:
    Description: 3.05 Ensure a log metric filter and alarm exist for CloudTrail configuration changes (Scored)
    Type: String
    Default: CIS Benchmark v.1.1 - 3.05 CloudTrail Config Changes
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  36ConsoleAuthFailures:
    Description: 3.06 Ensure a log metric filter and alarm exist for AWS Management Console authentication failures (Scored)
    Type: String
    Default: CIS Benchmark v.1.1 - 3.06 AWS Management Console Authentication Failures
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  37DisableDeleteCMK:
    Description: 3.07 Ensure a log metric filter and alarm exist for disabling or scheduled deletion of customer created CMKs (Scored)
    Type: String
    Default: CIS Benchmark v.1.1 - 3.07 Disabling or Scheduled Deletion of Customer Created CMKs
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  38S3BucketPolicyChanges:
    Description: 3.08 Ensure a log metric filter and alarm exist for S3 bucket policy changes (Scored)
    Type: String
    Default: CIS Benchmark v.1.1 - 3.08 S3 Bucket Policy Changes
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  39AWSConfigChanges:
    Description: 3.09 Ensure a log metric filter and alarm exist for AWS Config configuration changes (Scored)
    Type: String
    Default: CIS Benchmark v.1.1 - 3.09 AWS Config Configuration Changes
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  310SecGroupChanges:
    Description: 3.10 Ensure a log metric filter and alarm exist for AWS Config configuration changes (Scored)
    Type: String
    Default: CIS Benchmark v.1.1 - 3.10 Security Group Changes
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  311NACLChanges:
    Description: 3.11 Ensure a log metric filter and alarm exist for changes to Network Access Control Lists (NACL) (Scored)
    Type: String
    # Be careful - "()" not allowed so (NACL) becomes NACL.
    Default: CIS Benchmark v.1.1 - 3.11 Changes to Network Access Control Lists NACL
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  312NetworkGatewayChanges:
    Description: 3.12 Ensure a log metric filter and alarm exist for changes to network gateways (Scored)
    Type: String
    Default: CIS Benchmark v.1.1 - 3.12 Changes to Network Gateways
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  313RouteTableChanges:
    Description: 3.13 Ensure a log metric filter and alarm exist for route table changes (Scored)
    Type: String
    Default: CIS Benchmark v.1.1 - 3.13 Route Table Changes
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"
  
  314VPCChanges:
    Description: 3.14 Ensure a log metric filter and alarm exist for VPC changes (Scored)
    Type: String
    Default: CIS Benchmark v.1.1 - 3.14 VPC Changes
    MinLength: '1'
    MaxLength: '255'
    AllowedPattern: "[\\w\\s-_/.]+"
    ConstraintDescription: "Metric names can contain up to 255 alphanumeric characters, with the following also allowed: -_./"

Conditions:
  
  # Set these little monkeys to FALSE if no input from the user.
  NeedCloudTrailS3Bucket: !Equals [!Ref PreExistingS3BucketCloudTrail, ""]
  NeedCrossAccount01: !Equals [!Ref CrossAccount01, ""]
  NeedCrossAccount02: !Equals [!Ref CrossAccount02, ""]
  NeedCrossAccount03: !Equals [!Ref CrossAccount03, ""]
  WantALARMTopicName: !Equals [!Ref SetALARMTopicName, ""]
  NeedTRAILTopicName: !Equals [!Ref SetTRAILTopicName, ""]
  WantCloudTrailRoleName: !Equals [!Ref NameCloudTrailRole, ""]

Resources:
  
  CloudTrailS3Bucket:
    Condition: NeedCloudTrailS3Bucket
    DeletionPolicy: Retain
    Type: AWS::S3::Bucket
    Properties:
      # BucketName not required but auto-generated one looks sloppy. 
      # Reference: http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html#cfn-s3-bucket-name
      BucketName: !If [NeedCloudTrailS3Bucket, !Ref "AWS::NoValue", !Ref PreExistingS3BucketCloudTrail]
  
  CloudTrailS3BucketPolicy:
    #Condition: NeedCloudTrailS3Bucket
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !If [NeedCloudTrailS3Bucket, !Ref CloudTrailS3Bucket, !Ref PreExistingS3BucketCloudTrail]
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Sid: AWSCloudTrailAclCheck
          Effect: Allow
          Principal:
            Service: cloudtrail.amazonaws.com
          Action: s3:GetBucketAcl
          Resource:
            Fn::Join:
            - ''
            - - 'arn:aws:s3:::'
              - !If [NeedCloudTrailS3Bucket, !Ref CloudTrailS3Bucket, !Ref PreExistingS3BucketCloudTrail]
        - Sid: AWSCloudTrailWriteAdminS3
          Effect: Allow
          Principal:
            Service: cloudtrail.amazonaws.com
          Action: s3:PutObject
          Resource:
            - Fn::Join:
              - ''
              - - 'arn:aws:s3:::'
                - !If [NeedCloudTrailS3Bucket, !Ref CloudTrailS3Bucket, !Ref PreExistingS3BucketCloudTrail]
                - "/AWSLogs/"
                - Ref: AWS::AccountId
                - "/*"
            - Fn::Join:
              - ''
              - - 'arn:aws:s3:::'
                - !If [NeedCloudTrailS3Bucket, !Ref CloudTrailS3Bucket, !Ref PreExistingS3BucketCloudTrail]
                - "/AWSLogs/"
                # Not sure if this is the best way, but use default account if not CrossAccount specified.
                - !If [NeedCrossAccount01, !Ref "AWS::AccountId", !Ref CrossAccount01]
                - "/*"
            - Fn::Join:
              - ''
              - - 'arn:aws:s3:::'
                - !If [NeedCloudTrailS3Bucket, !Ref CloudTrailS3Bucket, !Ref PreExistingS3BucketCloudTrail]
                - "/AWSLogs/"
                - !If [NeedCrossAccount02, !Ref "AWS::AccountId", !Ref CrossAccount02]
                - "/*"
            - Fn::Join:
              - ''
              - - 'arn:aws:s3:::'
                - !If [NeedCloudTrailS3Bucket, !Ref CloudTrailS3Bucket, !Ref PreExistingS3BucketCloudTrail]
                - "/AWSLogs/"
                - !If [NeedCrossAccount03, !Ref "AWS::AccountId", !Ref CrossAccount03]
                - "/*"
          Condition:
            StringEquals:
              s3:x-amz-acl: bucket-owner-full-control
  
  TrailLogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      RetentionInDays:
        Ref: CloudWatchLogsRetention
  
  TrailLogGroupRole:
    Type: AWS::IAM::Role
    Properties:
      # RoleName not required but looks sloppy if not set.
      # If the user didn't input a name (i.e. FALSE condition), "AWS::NoValue" removes the RoleName property and a random name for the IAM role will be created.
      RoleName: !If [WantCloudTrailRoleName, !Ref "AWS::NoValue", !Ref NameCloudTrailRole]
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Sid: AssumeRole1
          Effect: Allow
          Principal:
            Service: cloudtrail.amazonaws.com
          Action: sts:AssumeRole
      Policies:
      - PolicyName: cloudtrail-policy
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Sid: AWSCloudTrailCreateLogStream2014110
            Effect: Allow
            Action:
            - logs:CreateLogStream
            Resource:
            - Fn::GetAtt:
              - TrailLogGroup
              - Arn
          - Sid: AWSCloudTrailPutLogEvents20141101
            Effect: Allow
            Action:
            - logs:PutLogEvents
            Resource:
            - Fn::GetAtt:
              - TrailLogGroup
              - Arn
  
  TrailTopic:
    Type: AWS::SNS::Topic
    Properties:
      DisplayName:
        Ref: SetTRAILTopicDisplayName
      TopicName: !If [NeedTRAILTopicName, !Ref "AWS::NoValue", !Ref SetTRAILTopicName]
  
  TrailTopicPolicy:
    DependsOn:
    - TrailTopic
    Type: AWS::SNS::TopicPolicy
    Properties:
      PolicyDocument:
        Version: '2008-10-17'
        Statement:
        - Sid: AWSCloudTrailSNSPolicy
          Effect: Allow
          Principal:
            Service: cloudtrail.amazonaws.com
          Resource:
            Ref: TrailTopic
          Action: sns:Publish
      Topics:
      - Ref: TrailTopic
  
  Trail:
    DependsOn:
    - CloudTrailS3BucketPolicy
    - TrailTopicPolicy
    Type: AWS::CloudTrail::Trail
    Properties:
      IncludeGlobalServiceEvents: true
      IsLogging: true
      IsMultiRegionTrail: true
      # If the user didn't input a name (i.e. FALSE condition), use the auto-generated bucket name created in resource name CloudTrailS3Bucket. Cannot use "AWS::NoValue" because property S3BucketName is required. Ref: http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-cloudtrail-trail.html#cfn-cloudtrail-trail-s3bucketname
      S3BucketName: !If [NeedCloudTrailS3Bucket, !Ref CloudTrailS3Bucket, !Ref PreExistingS3BucketCloudTrail]
      CloudWatchLogsLogGroupArn:
        Fn::GetAtt:
        - TrailLogGroup
        - Arn
      CloudWatchLogsRoleArn:
        Fn::GetAtt:
        - TrailLogGroupRole
        - Arn
      SnsTopicName:
        Fn::GetAtt:
        - TrailTopic
        - TopicName
  
  EmailAlarmNotificationTopic:
    Type: AWS::SNS::Topic
    Properties:
      DisplayName:
        Ref: SetALARMTopicDisplayName
      TopicName: !If [WantALARMTopicName, !Ref "AWS::NoValue", !Ref SetALARMTopicName]
      Subscription:
      # Reference for the different type of endpoints available: http://docs.aws.amazon.com/sns/latest/api/API_Subscribe.html
      - Endpoint: !Ref SetEmail
        Protocol: email

# This section configures Metric Filters and associated Alarms.
# Note that Metric Filter names cannot be set from CloudFormation.
# Ref: http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-logs-metricfilter.html
# Metric Filter Names will display as <stack name>-<resource ID>-<random number>
# Eg. cis-controls-3-monitoring-31UnAuthAPICallsMetricFilter-1GNOXICLU4ZZD
  31UnAuthAPICallsMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ ($.errorCode = \"*UnauthorizedOperation\") || ($.errorCode = \"AccessDenied*\" ) }"
      MetricTransformations:
      - MetricName: !Ref 31UnauthAPICalls
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  31UnAuthAPICallsAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 31UnauthAPICalls
      AlarmDescription: Alarm for Unauthorized API Calls.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 31UnauthAPICalls
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  32MgmtConsoleNoMFAMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ ($.additionalEventData.MFAUsed = \"No\") && ($.eventName = \"ConsoleLogin\") }"
      MetricTransformations:
      - MetricName: !Ref 32MgmtConsoleNoMFA
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  32MgmtConsoleNoMFAAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 32MgmtConsoleNoMFA
      AlarmDescription: Alarm for Management Console Sign-in without MFA .
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 32MgmtConsoleNoMFA
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  33UseOfRootAcctMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ $.userIdentity.type = \"Root\" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != \"AwsServiceEvent\" }"
      MetricTransformations:
      - MetricName: !Ref 33UseOfRootAcct
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  33UseOfRootAcctAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 33UseOfRootAcct
      AlarmDescription: Alarm for Usage of "root" Account.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 33UseOfRootAcct
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  34IAMPolicyChangesMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ ($.eventName=DeleteGroupPolicy) || ($.eventName=DeleteUserPolicy)
        || ($.eventName=PutGroupPolicy) || ($.eventName=PutRolePolicy) || ($.eventName=PutUserPolicy)
        || ($.eventName=CreatePolicy) || ($.eventName=DeletePolicy) || ($.eventName=CreatePolicyVersion)
        || ($.eventName=DeletePolicyVersion)  || ($.eventName=AttachRolePolicy) }"
      MetricTransformations:
      - MetricName: !Ref 34IAMPolicyChanges
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  34IAMPolicyChangesAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 34IAMPolicyChanges
      AlarmDescription: Alarm for IAM Policy Changes.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 34IAMPolicyChanges
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  35CloudTrailConfigChangesMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ ($.eventName = CreateTrail) || ($.eventName = UpdateTrail)
        || ($.eventName = DeleteTrail) || ($.eventName = StartLogging) || ($.eventName
        = StopLogging) }"
      MetricTransformations:
      - MetricName: !Ref 35CloudTrailConfigChanges
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  35CloudTrailConfigChangesAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 35CloudTrailConfigChanges
      AlarmDescription: Alarm for CloudTrail Configuration Changes.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 35CloudTrailConfigChanges
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  36ConsoleAuthFailuresMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: '{ ($.eventName = ConsoleLogin) && ($.errorMessage = "Failedauthentication")
        }'
      MetricTransformations:
      - MetricName: !Ref 36ConsoleAuthFailures
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  36ConsoleAuthFailuresAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 36ConsoleAuthFailures
      AlarmDescription: Alarm for Failed Console Logins.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 36ConsoleAuthFailures
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '10'
  
  37DisableDeleteCMKMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ ($.eventSource=kms.amazonaws.com) && (($.eventName=DisableKey)
        || ($.eventName=ScheduleKeyDeletion)) }"
      MetricTransformations:
      - MetricName: !Ref 37DisableDeleteCMK
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  37DisableDeleteCMKAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 37DisableDeleteCMK
      AlarmDescription: Alarm for Disabling or Scheduled Deletion of Customer Created CMKs.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 37DisableDeleteCMK
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  38S3BucketPolicyChangesMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ ($.eventSource=s3.amazonaws.com) && (($.eventName=PutBucketAcl)
        || ($.eventName = PutBucketPolicy) || ($.eventName = PutBucketCors) || ($.eventName
        = PutBucketLifecycle) || ($.eventName = PutBucketReplication) || ($.eventName
        = DeleteBucketPolicy) || ($.eventName = DeleteBucketCors) || ($.eventName=DeleteBucketLifecycle)
        || ($.eventName = DeleteBucketReplication)) }"
      MetricTransformations:
      - MetricName: !Ref 38S3BucketPolicyChanges
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  38S3BucketPolicyChangesAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 38S3BucketPolicyChanges
      AlarmDescription: Alarm for S3 Bucket Policy Changes.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 38S3BucketPolicyChanges
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  39AWSConfigChangesMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{($.eventSource = config.amazonaws.com) && (($.eventName=StopConfigurationRecorder)||($.eventName=DeleteDeliveryChannel)||($.eventName=PutDeliveryChannel)||($.eventName=PutConfigurationRecorder))}"
      MetricTransformations:
      - MetricName: !Ref 39AWSConfigChanges
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  39AWSConfigChangesAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 39AWSConfigChanges
      AlarmDescription: Alarm for AWS Config Configuration Changes.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 39AWSConfigChanges
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  310SecGroupChangesMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ ($.eventName = AuthorizeSecurityGroupIngress) || ($.eventName
        = AuthorizeSecurityGroupEgress) || ($.eventName = RevokeSecurityGroupIngress)
        || ($.eventName = RevokeSecurityGroupEgress) || ($.eventName = CreateSecurityGroup)
        || ($.eventName = DeleteSecurityGroup)}"
      MetricTransformations:
      - MetricName: !Ref 310SecGroupChanges
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  310SecGroupChangesAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 310SecGroupChanges
      AlarmDescription: Alarm for Security Group Changes.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 310SecGroupChanges
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  311NACLChangesMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ ($.eventName = CreateNetworkAcl) || ($.eventName = CreateNetworkAclEntry)
        || ($.eventName = DeleteNetworkAcl) || ($.eventName = DeleteNetworkAclEntry)
        || ($.eventName = ReplaceNetworkAclEntry) || ($.eventName = ReplaceNetworkAclAssociation)
        }"
      MetricTransformations:
      - MetricName: !Ref 311NACLChanges
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  311NACLChangesAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 311NACLChanges
      AlarmDescription: Alarm for Changes to Network Access Control Lists (NACL).
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 311NACLChanges
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  312NetworkGatewayChangesMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ ($.eventName = CreateCustomerGateway) || ($.eventName = DeleteCustomerGateway)
        || ($.eventName = AttachInternetGateway) || ($.eventName = CreateInternetGateway)
        || ($.eventName = DeleteInternetGateway) || ($.eventName = DetachInternetGateway)
        }"
      MetricTransformations:
      - MetricName: !Ref 312NetworkGatewayChanges
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  312NetworkGatewayChangesAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 312NetworkGatewayChanges
      AlarmDescription: Alarm for Changes to Network Gateways.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 312NetworkGatewayChanges
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  313RouteTableChangesMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ ($.eventName = CreateRoute) || ($.eventName = CreateRouteTable)
        || ($.eventName = ReplaceRoute) || ($.eventName = ReplaceRouteTableAssociation)
        || ($.eventName = DeleteRouteTable) || ($.eventName = DeleteRoute) || ($.eventName
        = DisassociateRouteTable) }"
      MetricTransformations:
      - MetricName: !Ref 313RouteTableChanges
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  313RouteTableChangesAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 313RouteTableChanges
      AlarmDescription: Alarm for Route Table Changes.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 313RouteTableChanges
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'
  
  314VPCChangesMetricFilter:
    DependsOn:
    - TrailLogGroup
    Type: AWS::Logs::MetricFilter
    Properties:
      LogGroupName:
        Ref: CloudTrailLogGroupName
      FilterPattern: "{ ($.eventName = CreateVpc) || ($.eventName = DeleteVpc) ||
        ($.eventName = ModifyVpcAttribute) || ($.eventName = AcceptVpcPeeringConnection)
        || ($.eventName = CreateVpcPeeringConnection) || ($.eventName = DeleteVpcPeeringConnection)
        || ($.eventName = RejectVpcPeeringConnection) || ($.eventName = AttachClassicLinkVpc)
        || ($.eventName = DetachClassicLinkVpc) || ($.eventName = DisableVpcClassicLink)
        || ($.eventName = EnableVpcClassicLink) }"
      MetricTransformations:
      - MetricName: !Ref 314VPCChanges
        MetricNamespace: !Ref NameYourNameSpace
        MetricValue: '1'
  
  314VPCChangesChangesAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: !Ref 314VPCChanges
      AlarmDescription: Alarm for VPC Changes.
      AlarmActions:
      - Ref: EmailAlarmNotificationTopic
      MetricName: !Ref 314VPCChanges
      Namespace: !Ref NameYourNameSpace
      ComparisonOperator: GreaterThanOrEqualToThreshold
      EvaluationPeriods: '1'
      Period: '300'
      Statistic: Sum
      Threshold: '1'

Outputs:
  CloudTrailS3BucketName:
    Description: CloudTrail S3 Target Bucket Name
    Value: !If [NeedCloudTrailS3Bucket, !Ref CloudTrailS3Bucket, !Ref PreExistingS3BucketCloudTrail]
    Export:
      Name: !Sub "${AWS::StackName}-BucketName"
  NotificationEmail:
    Description: Email for Alarms
    Value: !Ref SetEmail
  TrailLogGroupRole:
    Description: IAM Role Name for CloudTrail Logs
    Value: !GetAtt TrailLogGroupRole.Arn
```
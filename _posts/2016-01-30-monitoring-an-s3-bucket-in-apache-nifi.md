---
layout: post
title: Monitoring An S3 Bucket in Apache NiFi
---

# Introduction to NiFi

[Apache NiFi](http://nifi.apache.org/) is a relatively new data processing system with a plethora of general-purpose processors and a point and click interface. You are going to love it!

Processors in NiFi are like little black boxes of functionality. They're generally focused on doing one task, and can be chained together in powerful ways to accomplish arbitrary tasks. In a way, processors are like simple unix utilities that are piped together to produce the desired output, except NiFi understands much more complex interactions than mostly linear pipes.

NiFi can pull data from a variety of sources. Just starting out, one might use NiFi to process files on the local filesystem, or maybe files from a remote system via FTP/SFTP. NiFi supports a growing list of other transports, like GetHTTP, GetHDFS, GetKafka, GetMongo, GetJMS, GetSolr, and even GetTwitter.

# Using S3

Surprisingly, as of this writing, there is no GetS3. Initially this might seem like an oversight, but I don't think that is the case. In NiFi, although processors are single-purpose, Get* processors often (but not always) do at least three actions:

1. List the discrete units at the source (e.g. the files in a directory)
2. Copy the data into NiFi's internal content repository
3. Delete the data at the source.

This is how NiFi ensures data is processed only once without keeping state about which data has already been seen. For example, when using the GetFile processor, files are deleted from the local directory after being copied into NiFi. This behavior can be changed, but its the default, and for good reason.

If you instead don't want to list an entire directory, but would rather pick a particular file (the name of which you already know), you should use FetchFile. The file can still be deleted after the fetch if necessary, but there is no listing step.

NiFi currently ships with FetchS3Object, which is similar in concept to FetchFile.

Likewise, the ListFile processor will get you the listing of a directory and doesn't perform any other actions. Wouldn't a [ListS3](https://issues.apache.org/jira/browse/NIFI-840) processor fit the bill? This would be great for one-time actions, but not necessarily ideal for the ongoing monitoring of an S3 bucket.

If you've ever listed an Amazon S3 bucket, you know it can be a sensitive operation. Listing a single large bucket might take hours. And if you only care about new files posted to the bucket since your last listing, you'll have to keep track of a lot of state. Possibly hundreds of thousands of files.

Unfortunately, some of the nuances around these design decisions aren't immediately apparent when using NiFi, and it looks like the S3 support is incomplete. NiFi could include a GetS3 processor, but constant listing of the bucket could prove awkward, expensive (LIST operations are billed at a higher tier), not friendly to clusters, and not in line with common use cases. While some organizations might use an S3 bucket as a staging area or queue, my experience is that data is more often dropped into S3 and left there until expiration or deleted by a user action.


# SQS Notifications

Another way to monitor an S3 bucket for new files is to use notifications. AWS supports a few ways of doing this, but I'll focus on using SQS.

First, create an SQS queue in the [AWS Web console](https://console.aws.amazon.com/sqs/home). Queue creation should be fairly straightforward; you need to name it and can probably accept the defaults.

Next, the queue needs to be configured to allow event notifications from the bucket. Every time an event occurs in the bucket, a notification will be sent to the SQS queue. We have to tell SQS that our bucket is allowed to send notification of these events to our SQS queue. [The AWS notification docs](http://docs.aws.amazon.com/AmazonS3/latest/dev/NotificationHowTo.html#grant-destinations-permissions-to-s3) describe this in detail.

In the web console, select the newly created SQS queue, and choose the _Permissions_ tab below. The interface to edit policies seems to change (and improve) every time I use it, but makes it difficult to write instructions that will last. I've had success using this policy, which came directly from the AWS notification docs:

<pre>
<code>
{
 "Version": "2008-10-17",
 "Id": "example-ID",
 "Statement": [
  {
   "Sid": "example-statement-ID",
   "Effect": "Allow",
   "Principal": {
     "AWS": "*"  
   },
   "Action": [
    "SQS:SendMessage"
   ],
   "Resource": "SQS-ARN",
   "Condition": {
      "ArnLike": {          
      "aws:SourceArn": "arn:aws:s3:*:*:bucketname"    
    }
   }
  }
 ]
}
</code>
</pre>

Be sure to change the `bucketname` as necessary. If you have other existing policies for this SQS queue, be careful!

Once the policy is entered, the AWS console presents a summary like this:

![SQS Policy]({{ site.url }}/img/sqs-policy.png)

# Configure Events On The S3 Bucket

Now that the bucket permissions are configured, open the [S3 console](https://console.aws.amazon.com/s3/home), select the desired bucket, open the _Events_ section, and choose _Add Notification_. In the image below, we notify on all ObjectCreated events and send to an SQS queue (vs an SNS topic).

![Adding a bucket event]({{ site.url }}/img/bucket-events.png)

Save the event notification - the web console won't let you save if the SQS permissions are incorrect.

From this point forward, a new message will be put on the SQS queue each time an object is created in the S3 bucket.

# Creating The Graph

Now we just have to tell NiFi where to look. The flow is GetSQS -> SplitJSON -> ExtractText -> FetchS3Object -> UpdateAttribute -> PutFile.

![NiFi Flow with GetSQS]({{ site.url }}/img/nifi-sqs-flow.png)

### GetSQS: Grabs JSON messages off the queue
Each JSON message describes a bucket event. Configure the `Queue URL` (can be found in the AWS console) and AWS credentials

### SplitJSON: Extracts the S3 object key from the JSON message
Configure `JsonPath Expression` to `$.Records[*].s3.object.key`

### ExtractText: Sets the S3 object key as the filename attribute
Configure a new property called `filename` with the regex `(.*)`

### FetchS3Object: Download the object from S3
Configure the `Object Key` with `${filename}`

### UpdateAttribute: Change the S3 object key from path/to/key to path-to-key
Configure a new property called `filename` as `${filename:replaceAll("/", "-")}`. This replaces all slashes with dashes because the PutFile processor doesn't behave well with slashes

### PutFile: Copy the object to the local filesystem
Configure the `Directory` as desired

# NiFi Template

I've created a [NiFi template]({{ site.url }}/uploads/Monitor_S3_Bucket.xml) to aid creation of this flow.

# Conclusion

NiFi provides a nice set of tools to work with data on S3. I've been using this technique for a few months on NiFi 0.3.0 to ingest data from a low volume S3 bucket (a few files per minute), but it should scale to larger volumes nicely.

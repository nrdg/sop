# Amazon Web Services

We use Amazon Web Services for compute and storage.

## Batch computing with Cloudknot

We use [Cloudknot](https://nrdg.github.io/cloudknot/) for batch computing. Use of CK requires additional permissions, so you will
have to ask an administrator to provide these permissions, before you can run anything.

Always use spot instances with several retries.

## "Cold" storage in Glacier

When a dataset is not routinely accessed, it should be moved to cold storage in [Glacier](https://aws.amazon.com/glacier/)

Moving an entire bucket to AWS can be done using a recursive self-copy with change of storage class:

    aws s3 cp s3://<bucket-name>/ s3://<bucket-name>/ --recursive --storage-class GLACIER

On the other hand, moving data out of Glacier is a bit [trickier](https://aws.amazon.com/premiumsupport/knowledge-center/restore-s3-object-glacier-storage-class/), requiring either use of the [`s3cmd`](https://s3tools.org/s3cmd) tool:

    s3cmd restore --recursive s3://mybucketname/folder/ --restore-days=NUM --restore-priority=bulk

(see [here](https://stackoverflow.com/a/27990436/3532933)) or [slightly more elaborate use of the aws CLI](https://stackoverflow.com/a/21711321/3532933):

    aws s3 ls s3://<bucket_name> | awk '{print $4}' | xargs -L 1 aws s3api restore-object --restore-request Days=<days> --bucket <bucket_name> --key

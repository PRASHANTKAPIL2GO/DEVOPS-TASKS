# mapp-dr-bucket — Amazon S3 Disaster Recovery Bucket

## Overview

`mapp-dr-bucket` is an Amazon S3 general-purpose bucket hosted in the **Europe (Stockholm)** region (`eu-north-1`), used as a disaster recovery (DR) storage target. It is managed under the AWS account **MANAS (660815084882)** via the `terraform-prashant` IAM role.

---

## Bucket Details

| Property        | Value                              |
|-----------------|------------------------------------|
| **Bucket Name** | `mapp-dr-bucket`                   |
| **ARN**         | `arn:aws:s3:::mapp-dr-bucket`      |
| **Region**      | Europe (Stockholm) — `eu-north-1`  |
| **Storage Class** | Standard                         |
| **Account**     | MANAS (660815084882)               |

---

## Current Contents

| File Name              | Type | Size     | Last Modified                  | Storage Class |
|------------------------|------|----------|--------------------------------|---------------|
| `Screenshot (28).png`  | png  | 356.3 KB | March 1, 2026, 13:46:20 (UTC+05:30) | Standard |

> Objects are visible under the **Objects** tab in the S3 console.

---

## Bucket Policy

The bucket has a custom IAM policy applied. Below is the policy currently configured:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPutObjectOnlyInUploads",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-policy-bucket-2025/uploads/*"
    },
    {
      ...
    }
  ]
}
```

> **Note:** The first statement (`AllowPutObjectOnlyInUploads`) restricts `PutObject` access to the `uploads/` prefix of a separate bucket (`my-policy-bucket-2025`). Ensure the full policy (including any additional statements) is reviewed and applied correctly for this DR bucket.

---

## Access & Permissions

- The bucket is accessed via the **`terraform-prashant`** IAM role.
- Permissions are managed through the bucket policy (visible under the **Permissions** tab).
- For external access, permissions must be explicitly granted — objects are **not publicly accessible** by default.

---

## Usage

### Uploading Objects

Use the AWS Console **Upload** button, AWS CLI, or Terraform:

```bash
# AWS CLI example
aws s3 cp ./your-file.ext s3://mapp-dr-bucket/ --region eu-north-1
```

### Listing Objects

```bash
aws s3 ls s3://mapp-dr-bucket/ --region eu-north-1
```

### Downloading Objects

```bash
aws s3 cp s3://mapp-dr-bucket/Screenshot\ \(28\).png ./ --region eu-north-1
```

---

## Notes

- This bucket is intended for **disaster recovery** purposes — treat stored objects accordingly.
- Review and restrict the bucket policy `Principal: "*"` if public write access is not intended.
- Enable **versioning** under the Properties tab if point-in-time recovery is required.
- Consider enabling **S3 Lifecycle rules** to transition older objects to cheaper storage classes (e.g., Glacier) for cost optimisation.

---

## Related Resources

- [Amazon S3 Documentation](https://docs.aws.amazon.com/s3/)
- [S3 Bucket Policy Examples](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-bucket-policies.html)
- [S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
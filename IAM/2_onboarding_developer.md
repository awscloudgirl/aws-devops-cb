# Use Case: Onboarding a Developer and Granting Specific AWS Service Access to CloudBinary AWS Account

This document provides a step-by-step guide to onboard a new developer (e.g., **Jack**) and grant them specific AWS service access within the **CloudBinary** AWS account. The process includes creating a user group, adding the user, creating and attaching a custom IAM policy, and validating access.

---

## Table of Contents

- [Step 1: Create a Group](#step-1-create-a-group)
- [Step 2: Create a User](#step-2-create-a-user)
- [Step 3: Add User to Group](#step-3-add-user-to-group)
- [Step 4: Create a Custom IAM Policy](#step-4-create-a-custom-iam-policy)
- [Step 5: Attach Policy to Group](#step-5-attach-policy-to-group)
- [Step 6: Validate Access as Jack](#step-6-validate-access-as-jack)
- [Additional Notes](#additional-notes)

---

## Step 1: Create a Group

Create a new IAM group for UI developers.

- **Group Name**: `uideveloper`

**Action**: In the AWS IAM console, navigate to **User Groups** and click **Create group**.

![Create_Group](images/step1_create_group.png)

---

## Step 2: Create a User

Create a new IAM user for the developer.

- **User Name**: `jack`
- **Access Type**:
  - ✅ Programmatic access (for CLI/SDK access)
  - ✅ AWS Management Console access (for Console access)
- **Console Password**: Set a custom password or auto-generate one.

**Action**: Go to **Users** in the IAM console and click **Add user**.

![Create_Group](images/step2_create_user.png)

**Note**: Provide the following details to Jack securely:

- **Console sign-in URL**: `https://<your-account-id>.signin.aws.amazon.com/console`
- **User name**: `jack`
- **Console password**: (Provide the initial password or instructions to set one)

---

## Step 3: Add User to Group

Add the user `jack` to the `uideveloper` group.

**Action**: After creating the user, on the **Add user to group** step, select the `uideveloper` group.

![Create_Group](images/step3_add_user_to_group.png)

---

## Step 4: Create a Custom IAM Policy

Create a customer-managed IAM policy with specific AWS service access.

**Policy Name**: `uideveloper-policy`

**Action**:

1. Navigate to **Policies** in the IAM console and click **Create policy**.
2. Choose the **JSON** tab and paste the policy below.

**Policy JSON**:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:*",
                "rds:*",
                "s3:*",
                "dynamodb:*",
                "lambda:*",
                "apigateway:*",
                "states:*",  // Step Functions
                "ses:*",
                "sns:*",
                "codebuild:*",
                "codecommit:*",
                "codepipeline:*"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestedRegion": [
                        "us-east-1",   // Specify allowed regions
                        "us-west-2"
                    ]
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "iam:PassedToService": [
                        "ec2.amazonaws.com",
                        "lambda.amazonaws.com",
                        "apigateway.amazonaws.com",
                        "states.amazonaws.com",
                        "codebuild.amazonaws.com",
                        "codepipeline.amazonaws.com"
                        // Add other services if needed
                    ],
                    "aws:RequestedRegion": [
                        "us-east-1",
                        "us-west-2"
                    ]
                }
            }
        }
    ]
}
```

**Note**: Adjust the regions (`aws:RequestedRegion`) as per your requirements.

---

## Step 5: Attach Policy to Group

Attach the custom IAM policy to the `uideveloper` group.

**Action**:

1. In the IAM console, go to **User Groups**.
2. Select the `uideveloper` group.
3. Under the **Permissions** tab, click **Add permissions** -> **Attach policies**.
![Attach_Policy](images/step4_attach_policy.png)
4. Search for `uideveloper-policy` and attach it.
![Attach_Policy](images/step5_attach_policy.png)

**Screenshot Suggestion**: Show the policy being attached to the group.
- **Filename**: `step5_attach_policy.png` also did step4 permissions attach policies photo

---

## Step 6: Validate Access as Jack

As the user `jack`, log in to the CloudBinary AWS account and validate access.

**Action**:

1. Provide Jack with the login credentials and the console sign-in URL.
2. Instruct Jack to:

   - Log in to the AWS Management Console.
   - Navigate to the allowed services (EC2, RDS, S3, etc.) in the permitted regions (`us-east-1`, `us-west-2`).
   - Attempt to perform actions to confirm access.
   - Try accessing a service or region not specified in the policy to ensure access is denied.

**Screenshot Suggestion**:

![Attach_Policy](images/step6_validate_access.png)

![Attach_Policy](images/step6_access_denied.png)

---

## Additional Notes

- **Password Security**: Ensure that passwords are communicated securely and not stored in plaintext.
- **Region Restrictions**: The policy limits access to specific regions. Modify the `aws:RequestedRegion` array if you need to allow access to additional regions.
- **AWS Gov and China Regions**: Access to AWS GovCloud and China regions requires separate accounts and compliance with additional regulations.
- **Policy Updates**: If additional services are needed in the future, update the IAM policy accordingly.

---

By following these steps, you ensure that the developer has the necessary access to perform their tasks while maintaining the security and integrity of your AWS environment.

---

### Summary

This guide helps onboard a new developer by:

- Creating a dedicated IAM group.
- Setting up a user with specific access types.
- Attaching a custom IAM policy that grants access to required AWS services in specified regions.
- Validating that the user has appropriate access.

**Remember**: Regularly review IAM policies and access levels to adhere to the principle of least privilege.


# AWS CLI Installation Guide

This document provides instructions for installing the **AWS CLI** (Command Line Interface) on both **Windows** and **macOS**. The AWS CLI allows you to interact with AWS services using commands from your terminal or command prompt.

## Prerequisites

- A valid **AWS account**. If you don’t have one, you can sign up at [aws.amazon.com](https://aws.amazon.com/).
- Administrator access to your computer for installation.

---

## Table of Contents
- [Installing AWS CLI on Windows](#installing-aws-cli-on-windows)
- [Installing AWS CLI on macOS](#installing-aws-cli-on-macos)
- [Verify the Installation](#verify-the-installation)
- [Configure AWS CLI](#configure-aws-cli)
- [AWS STS get-caller-identity Command](#aws-sts-get-caller-identity-command)
- [Uninstalling AWS CLI](#uninstalling-aws-cli)

---

## Installing AWS CLI on Windows

To install the AWS CLI on a Windows machine, follow these steps:

### Step 1: Download the AWS CLI Installer
1. Go to the official AWS CLI download page: [AWS CLI for Windows](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
2. Download the **AWS CLI MSI installer** for Windows.

### Step 2: Run the Installer
1. Open the downloaded `.msi` file.
2. Follow the instructions in the setup wizard.
   - Accept the license agreement.
   - Choose the installation folder or keep the default.
   - Click **Install** and wait for the installation to complete.

### Step 3: Confirm the Installation
Once the installation is complete, proceed to the [Verify the Installation](#verify-the-installation) section to ensure the AWS CLI is working.

---

## Installing AWS CLI on macOS

To install the AWS CLI on macOS, follow these steps:

### Option 1: Using Homebrew (Recommended)

If you have **Homebrew** installed, the AWS CLI installation process is simple.

1. Open **Terminal**.
2. Run the following command to install AWS CLI using Homebrew:

   ```bash
   brew install awscli
   ```

3. After the installation completes, you can verify the installation by following the steps in the [Verify the Installation](#verify-the-installation) section.

### Option 2: Using the AWS Installer

1. Go to the official AWS CLI download page: [AWS CLI for macOS](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
2. Download the **AWS CLI ZIP installer** for macOS.
3. Extract the downloaded file:

   ```bash
   unzip awscli-exe-macos.zip
   ```

4. Run the installer:

   ```bash
   sudo ./aws/install
   ```

Once the installation completes, move on to the [Verify the Installation](#verify-the-installation) section.

---

## Verify the Installation

After installing the AWS CLI, confirm that the installation was successful by running the following command in **Command Prompt** (Windows) or **Terminal** (macOS):

```bash
aws --version
```

You should see output similar to the following:

```
aws-cli/2.x.x Python/3.x.x <platform> botocore/2.x.x
```

This confirms that the AWS CLI is installed correctly.

---

## Configure AWS CLI

To configure the AWS CLI for use, you'll need your **AWS Access Key ID**, **Secret Access Key**, **default region**, and **default output format**. Follow these steps:

1. Open a terminal or command prompt.
2. Run the following command to configure the AWS CLI:

   ```bash
   aws configure
   ```

3. You will be prompted to enter your AWS credentials:

   ```
   AWS Access Key ID [None]: YOUR_ACCESS_KEY
   AWS Secret Access Key [None]: YOUR_SECRET_KEY
   Default region name [None]: YOUR_REGION (e.g., us-east-1)
   Default output format [None]: json
   ```

   Replace the placeholders with your actual AWS credentials.

---

## AWS STS `get-caller-identity` Command

After installing and configuring the AWS CLI, you can use the `aws sts get-caller-identity` command to retrieve information about the IAM identity you're using to run AWS CLI commands.

### What Does `get-caller-identity` Do?

The `get-caller-identity` command returns details about the IAM user or role whose credentials are being used to make the request. This is helpful for verifying which user or role is being used for a particular AWS session.

### Example Command

Run the following command to get information about your current AWS identity:

```bash
aws sts get-caller-identity
```

### Example Output:

```json
{
  "UserId": "AIDACKCEVSQ6C2EXAMPLE",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/Alice"
}
```

- **UserId**: The unique identifier for the IAM user or role.
- **Account**: The AWS account ID.
- **Arn**: The Amazon Resource Name (ARN) of the IAM user or role.

### Use Case

You can use this command to verify which user or role is making the AWS requests. This is useful when working with multiple AWS accounts or roles, or when troubleshooting access issues.

---

## Uninstalling AWS CLI

### On Windows
1. Open **Control Panel**.
2. Go to **Programs > Uninstall a program**.
3. Select **AWS Command Line Interface** and click **Uninstall**.

### On macOS
To uninstall the AWS CLI, run the following command in the terminal:

```bash
sudo rm -rf /usr/local/aws-cli
sudo rm /usr/local/bin/aws
```

---

## Useful Commands

Here are some basic AWS CLI commands to get started:

- List all S3 buckets:

  ```bash
  aws s3 ls
  ```

- Describe all EC2 instances in your default region:

  ```bash
  aws ec2 describe-instances
  ```

- Check your AWS CLI configuration:

  ```bash
  aws configure list
  ```

For a full list of available AWS CLI commands, visit the [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/).

---

## Additional Resources

- [AWS CLI Official Documentation](https://docs.aws.amazon.com/cli/latest/userguide/)
- [AWS CLI GitHub Repository](https://github.com/aws/aws-cli)


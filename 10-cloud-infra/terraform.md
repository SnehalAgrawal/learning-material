# Cloud Architecture: Terraform (Infrastructure as Code)

### 1. Overview
Terraform is an open-source tool that allows you to define and provide data center infrastructure using a high-level configuration language (HCL). For senior engineers, Terraform is the standard for managing "Immutable Infrastructure," where changes are made by modifying code rather than manually clicking in a Cloud Console.

### 2. Key Concepts
*   **Providers**: Plugins that allow Terraform to interact with cloud providers (AWS, GCP, Azure), SaaS providers, or APIs.
*   **Resources**: The "What" of your infrastructure—e.g., an EC2 instance, an S3 bucket, or a VPC.
*   **Modules**: Containers for multiple resources that are used together. Modules allow you to create reusable "packages" of infrastructure (e.g., a "Standard Web Server" module).
*   **State**: Terraform keeps track of the metadata of your infrastructure in a `terraform.tfstate` file. This is the "Source of Truth" for what actually exists in the cloud.
*   **Plan & Apply**: The core workflow. `terraform plan` shows you what will happen, and `terraform apply` makes it happen.

### 3. Real-World Usage
*   **Multi-Cloud Strategy**: Using a single tool to manage resources across AWS and Google Cloud simultaneously.
*   **Infrastructure Versioning**: Storing your infrastructure code in Git, allowing you to see exactly how your network setup has changed over the last 6 months.
*   **Disaster Recovery**: Replicating an entire production environment in a different region within minutes by simply changing a variable and running `apply`.
*   **Drift Detection**: Running a "Plan" regularly to see if someone manually changed a setting in the AWS console (Manual changes = "Drift").

### 4. Tradeoffs
*   **Declarative vs. Imperative**: You define the *end state* you want (Declarative), and Terraform figures out how to get there. This is easier than writing scripts (Imperative) but can be tricky when resource dependencies are complex.
*   **State Management**: If the state file is lost or corrupted, Terraform "forgets" what it built, which can lead to orphaned resources or duplicate infrastructure.
*   **HCL Learning Curve**: While powerful, HashiCorp Configuration Language (HCL) is another language to learn, though it's more readable than JSON or YAML for infrastructure.

### 5. When NOT to Use
*   **One-off Experiments**: If you are just testing a single service for 10 minutes, clicking in the console is faster than writing HCL.
*   **Highly Dynamic Resources**: If resources are being created and destroyed every few seconds (like serverless functions scaling), a specialized framework (like Serverless Framework or AWS SAM) might be better.
*   **Small Teams with Zero Cloud Knowledge**: The overhead of managing state, backends, and locking might be too much for a team that only needs one server.

### 6. Interview Focus
*   **State Locking**: "Why is it important to use a remote backend with state locking?"
    *   To prevent multiple engineers from running `apply` at the same time and corrupting the state.
*   **Lifecycle**: "Explain what `prevent_destroy` or `create_before_destroy` does in a resource block."
    *   prevent_destroy is used to prevent the accidental deletion of a resource. create_before_destroy is used to create a new resource before destroying the old resource to avoid downtime.
*   **Refactoring**: "How do you move a resource from one module to another without Terraform trying to delete and recreate it?"
    *   `terraform state mv` or `moved` blocks.

### 7. Common Mistakes
*   **Secrets in State**: Committing the `terraform.tfstate` file to Git. State files often contain plain-text database passwords and API keys.
*   **Monolithic State**: Putting your entire company's infrastructure in one big folder. This makes `plan` slow and increases the "Blast Radius" if a mistake is made.
*   **Hardcoding Providers**: Not pinning provider versions, which can lead to breaking changes when HashiCorp releases a new version of the AWS provider.

### Example

#### 1. Define the Cloud Provider (AWS in this case)

```terraform
provider "aws" {
  region = "us-east-1"
}
```

#### 2. Define Infrastructure Resources (An S3 Bucket)
```terraform
resource "aws_s3_bucket" "example" {
  bucket = "my-unique-bucket-name"
}
```

#### 3. Output the Bucket URL
```terraform
output "bucket_url" {
  value = aws_s3_bucket.example.bucket_domain_name
}
```

#### For complete reference , here is the full terraform file to setup a simple web server

```terraform
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "example-server"
  }
}

output "instance_ip" {
  value = aws_instance.example.public_ip
}
```
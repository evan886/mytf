非常好 👍
你是 **Infra 工程师**，公司已经有 Terraform 项目用于管理 **AWS 基础设施**，你要的是一个**快速入门 + 实战导向**的教程，让你能马上看懂别人写的 Terraform 代码，并自己改、跑、调。

我们来一步步搞清楚 Terraform 的核心概念、命令流程和AWS实战（花 30–60 分钟搞明白）。

---

## 🚩 一、Terraform 是什么

Terraform 是一种 **基础设施即代码（Infrastructure as Code, IaC）** 工具，用来**定义、创建和管理云资源（AWS、Azure、GCP、K8s等）**。

你写的 `.tf` 文件定义了你要的基础设施，Terraform 通过 `terraform apply` 自动创建、修改或销毁这些资源。

👉 就像 **用代码管理 AWS Console 上的所有操作**。

---

## 🚩 二、核心概念（非常关键）

| 概念              | 含义                  | AWS 对应              |
| --------------- | ------------------- | ------------------- |
| **Provider**    | 告诉 Terraform 要操作哪个云 | 例如 `aws`            |
| **Resource**    | 你要创建的资源             | EC2, S3, IAM Role   |
| **Data Source** | 读取已有资源的数据           | 例如已有的 VPC           |
| **Variable**    | 变量，方便复用             | `${var.region}`     |
| **Output**      | 输出值                 | 比如打印出EC2 IP         |
| **State 文件**    | 保存当前资源状态            | `terraform.tfstate` |
| **Module**      | 模块化管理资源             | “可重用模板”             |

---

## 🚩 三、最小可运行例子（AWS 实战）

假设你要在 AWS 创建一个 EC2 实例：

```bash
mkdir tf-demo && cd tf-demo
```

**文件：main.tf**

```hcl
provider "aws" {
  region = "ap-southeast-1" # 新加坡区域
}

resource "aws_instance" "demo" {
  ami           = "ami-0c55b159cbfafe1f0" # Amazon Linux 2
  instance_type = "t3.micro"

  tags = {
    Name = "terraform-demo"
  }
}
```

---

## 🚩 四、Terraform 命令核心流程（记住这五个）

| 命令                  | 作用                |
| ------------------- | ----------------- |
| `terraform init`    | 初始化项目，下载 provider |
| `terraform plan`    | 预览执行计划（不会真正创建资源）  |
| `terraform apply`   | 执行创建资源            |
| `terraform show`    | 查看当前资源状态          |
| `terraform destroy` | 销毁资源              |

👣 实战演练：

```bash
terraform init
terraform plan
terraform apply -auto-approve
terraform show
terraform destroy -auto-approve
```

执行完后，你可以登录 AWS 控制台 → EC2 → 看到实例被创建 & 删除。

---

## 🚩 五、变量与模块化（看懂公司项目的关键）

公司 Terraform 项目一般不会把所有资源都写在 `main.tf`，而是拆成模块。

### 1️⃣ 变量定义

`variables.tf`

```hcl
variable "region" {
  default = "ap-southeast-1"
}
variable "instance_type" {
  default = "t3.micro"
}
```

### 2️⃣ 调用变量

`main.tf`

```hcl
provider "aws" {
  region = var.region
}

resource "aws_instance" "demo" {
  ami           = "ami-01c94064639c71719"
  instance_type = var.instance_type
}
```

### 3️⃣ 模块结构（你在公司看到的可能长这样）

```
terraform/
├── main.tf
├── variables.tf
├── outputs.tf
├── modules/
│   ├── vpc/
│   ├── ec2/
│   └── rds/
└── environments/
    ├── dev/
    └── prod/
```

主项目通过 `module` 引用：

```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr_block = "10.0.0.0/16"
}
```

模块内部再定义多个资源，比如子网、网关、路由表等。

---

## 🚩 六、State 文件与远程管理

Terraform 的“记忆”在 `.tfstate` 文件里。

* 本地运行：`terraform.tfstate` 在本地目录。
* 公司项目：一般配置在 **远程 backend**，例如 AWS S3 + DynamoDB，用于多人协作。

例子：

```hcl
terraform {
  backend "s3" {
    bucket = "company-terraform-state"
    key    = "prod/network.tfstate"
    region = "ap-southeast-1"
  }
}
```

---

## 🚩 七、看懂公司 Terraform 项目的技巧

1. 先打开 `main.tf` → 看哪些 **module** 被引用。
2. 打开 `terraform.tfvars` → 看部署时使用的变量值。
3. 搜索 `provider "aws"` → 看是哪个 region。
4. 执行 `terraform plan` → 模拟看看它会创建/修改哪些资源。
5. 如果用了 **workspace**（多环境），用：

   ```bash
   terraform workspace list
   terraform workspace select dev
   ```

   不同 workspace 会对应不同 tfstate（例如 dev/prod）。

---

## 🚩 八、常见坑（提前避雷）

| 问题                           | 原因                                   |
| ---------------------------- | ------------------------------------ |
| `Error acquiring state lock` | 远程 state 被别人占用（等一会或手动解锁）             |
| 修改手工 AWS 资源后，Terraform 不知道   | 因为 state 里没有更新，要用 `terraform import` |
| 不小心删除 .tfstate               | 会失联资源，要重新 import                     |
| region 写错，资源创建到错误区域          | provider 配置问题                        |

---

## 🚩 九、练手建议（1小时快速入门路线）

| 时间       | 内容                                 |
| -------- | ---------------------------------- |
| 0-15 分钟  | 跑上面 EC2 Demo                       |
| 15-30 分钟 | 改成使用变量                             |
| 30-45 分钟 | 增加一个 S3 bucket                     |
| 45-60 分钟 | 看公司项目目录结构，执行 `terraform plan` 观察变化 |

---

## 🚩 十、你能快速上手的关键目标

✅ 看懂模块化结构（module 概念）
✅ 会跑 init → plan → apply → destroy
✅ 能改变量、查看 state
✅ 明白 backend 在管理 state
✅ 能看懂别人写的 resource 定义

---

如果你愿意，我可以：

* 🔹 帮你快速解析公司 Terraform 项目的目录结构（你贴一段目录）
* 🔹 或者帮你写一个**公司级 AWS Terraform 模板（含S3、EC2、VPC、IAM）**的入门版

你想我帮你哪一个？（解析现有项目结构 vs 从零写一个公司风格的 AWS Terraform 模板）


# Terraform 快速入门教程

我帮你梳理一个快速上手的教程，重点讲解核心概念和实际应用。

## 一、核心概念理解

### 1. **什么是 Terraform？**
- **基础设施即代码（IaC）工具**：用代码管理云资源（EC2、RDS、VPC等）
- **声明式**：你描述"想要什么"，而不是"怎么做"
- **Provider**：连接云平台的插件（AWS、Azure、GCP等）

### 2. **核心文件结构**
```
project/
├── main.tf           # 主要资源定义
├── variables.tf      # 变量声明
├── outputs.tf        # 输出值
├── terraform.tfvars  # 变量赋值（不提交到git）
├── provider.tf       # Provider配置
└── backend.tf        # State存储配置
```

## 二、关键概念详解

### 1. **State（状态文件）**
```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
  }
}
```
- **作用**：记录当前基础设施的真实状态
- **重要性**：团队协作必须用远程 backend（S3）
- **不要手动编辑** state 文件！

### 2. **Resource（资源）**
```hcl
# 创建一个 EC2 实例
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}
```
- 格式：`resource "类型" "名称" { 配置 }`
- 引用方式：`aws_instance.web.id`

### 3. **Data Source（数据源）**
```hcl
# 查询已存在的 VPC
data "aws_vpc" "existing" {
  id = "vpc-12345678"
}

# 使用数据
resource "aws_subnet" "example" {
  vpc_id = data.aws_vpc.existing.id
}
```
- 用于**读取**已存在的资源
- 不会创建或修改资源

### 4. **Variables（变量）**
```hcl
# variables.tf
variable "instance_type" {
  description = "EC2 实例类型"
  type        = string
  default     = "t2.micro"
}

variable "environment" {
  type = string
}

# main.tf 中使用
resource "aws_instance" "web" {
  instance_type = var.instance_type
}
```

### 5. **Outputs（输出）**
```hcl
# outputs.tf
output "instance_ip" {
  description = "EC2 公网IP"
  value       = aws_instance.web.public_ip
}
```
- 显示资源的重要信息
- 可以被其他 Terraform 项目引用

### 6. **Modules（模块）**
```hcl
# 调用模块
module "vpc" {
  source = "./modules/vpc"
  
  cidr_block = "10.0.0.0/16"
  environment = "prod"
}

# 使用模块输出
resource "aws_instance" "web" {
  subnet_id = module.vpc.public_subnet_ids[0]
}
```
- 可复用的代码组件
- 类似编程中的函数

## 三、常用命令

```bash
# 1. 初始化（首次或添加新 provider 时）
terraform init

# 2. 格式化代码
terraform fmt

# 3. 验证配置
terraform validate

# 4. 预览变更（最常用！）
terraform plan

# 5. 应用变更
terraform apply

# 6. 查看当前状态
terraform show

# 7. 查看资源列表
terraform state list

# 8. 销毁资源
terraform destroy

# 9. 仅应用特定资源
terraform apply -target=aws_instance.web

# 10. 刷新状态
terraform refresh
```

## 四、实际工作流程

### 日常修改流程
```bash
# 1. 拉取最新代码
git pull

# 2. 修改 .tf 文件

# 3. 初始化（如有新 provider）
terraform init

# 4. 格式化
terraform fmt

# 5. 预览变更（重要！）
terraform plan

# 6. 确认无误后应用
terraform apply

# 7. 提交代码
git add .
git commit -m "Add new resource"
git push
```

### 阅读公司项目的步骤
```bash
# 1. 查看目录结构
tree -L 2

# 2. 先看这些文件
- provider.tf  # 了解用的哪些云平台
- backend.tf   # state 存在哪里
- variables.tf # 有哪些可配置参数

# 3. 查看资源列表
terraform state list

# 4. 查看具体资源
terraform state show aws_instance.web

# 5. 生成依赖关系图（可选）
terraform graph | dot -Tpng > graph.png
```

## 五、常见模式和最佳实践

### 1. 使用 locals 简化重复逻辑
```hcl
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
    Team        = "Infrastructure"
  }
}

resource "aws_instance" "web" {
  # ...
  tags = merge(local.common_tags, {
    Name = "WebServer"
  })
}
```

### 2. 使用 count 创建多个相似资源
```hcl
resource "aws_instance" "web" {
  count = 3
  
  ami           = "ami-xxx"
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-${count.index}"
  }
}
```

### 3. 使用 for_each 更灵活
```hcl
variable "instances" {
  default = {
    web  = "t2.micro"
    api  = "t2.small"
    db   = "t2.medium"
  }
}

resource "aws_instance" "servers" {
  for_each = var.instances
  
  instance_type = each.value
  tags = {
    Name = each.key
  }
}
```

## 六、常见陷阱和注意事项

### ⚠️ 重要提醒

1. **永远先 `plan` 再 `apply`**
   - plan 显示将要做什么变更
   - 看到 `-/+` 表示会先删除再创建（数据会丢失！）

2. **理解 State 的重要性**
   - State 丢失 = Terraform 不知道管理了什么
   - 不要手动修改 state
   - 不要把 state 提交到 git

3. **资源命名冲突**
   - AWS 某些资源名称全局唯一（S3 bucket）
   - 建议加前缀：`${var.project}-${var.environment}-bucket`

4. **Import 已存在的资源**
   ```bash
   # 如果资源已存在，可以导入
   terraform import aws_instance.web i-1234567890abcdef0
   ```

5. **依赖关系**
   ```hcl
   # 显式声明依赖
   resource "aws_instance" "web" {
     # ...
     depends_on = [aws_security_group.web]
   }
   ```

## 七、快速参考示例

### 完整的 AWS EC2 示例
```hcl
# provider.tf
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# variables.tf
variable "aws_region" {
  default = "us-east-1"
}

variable "environment" {
  default = "dev"
}

# main.tf
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_security_group" "web" {
  name = "${var.environment}-web-sg"
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.web.id]
  
  tags = {
    Name        = "${var.environment}-web"
    Environment = var.environment
  }
}

# outputs.tf
output "instance_public_ip" {
  value = aws_instance.web.public_ip
}
```

## 八、快速上手清单

- [ ] 理解 State 的作用
- [ ] 会用 `plan` 预览变更
- [ ] 理解 resource vs data source
- [ ] 会读懂变量和输出
- [ ] 知道模块的基本用法
- [ ] 会查看 `terraform state list`
- [ ] 了解常见资源的属性引用方式

现在你可以：
1. 先在公司项目里运行 `terraform plan` 看看输出
2. 找一个简单的资源（如 tag）尝试修改
3. 有问题随时查官方文档：https://registry.terraform.io/providers/hashicorp/aws/latest/docs

有具体问题随时问我！

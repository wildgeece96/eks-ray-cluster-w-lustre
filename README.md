# eks-ray-cluster-w-lustre
This is the sample code repository for EKS x Terraform x Ray x FSx for Lustre for distributed machine learning training.

## Setup 手順

S3 バケットを事前に作成しておく。  
バケット名を fsx-for-lustre.tf で記録する。  

`terraform/vpc.tf` の main.tf にて、azs を指定する箇所があるので Capacity Reservation の詳細画面から該当する AZ の値を
```terraform
locals {
  name   = var.name
  region = var.region
  azs    = ["us-west-1c"]  # こちらに穴埋めする
  tags = {
    Blueprint  = local.name
    GithubRepo = "github.com/awslabs/data-on-eks"
  }
}
```
`terraform/eks.tf` にて、キャパシティ予約 ID を確認して入力する。  
キャパシティ予約は下記 URL から確認が可能。  
https://us-west-1.console.aws.amazon.com/ec2/home?region=ap-northeast-1#CapacityReservations:  

```terraform
capacity_reservation_specification = {
    capacity_reservation_target = {
        capacity_reservation_id = "cr-0498091fd0522c240"
    }
    }

min_size     = 3
max_size     = 3
desired_size = 3
```

### VPC などのネットワーク周りのデプロイ  
```sh
export AWS_REGION=ap-northeast-1
terraform init
terraform plan \
-target=module.vpc \
-var 'region=us-west-1'
```
問題なければ、terraform apply
```sh
terraform apply \
-target=module.vpc \
-var 'region=us-west-1' \
--auto-approve
```

private subnet id の値を `terraform/eks.tf` で更新する。  
```terraform
subnet_ids = ["subnet-xxxxx"]  # ここの値

# If you are using a Capacity Reservation, the Subnet for the instances must match AZ for the reservation.
capacity_reservation_specification = {
capacity_reservation_target = {
    capacity_reservation_id = "cr-xxxxxxxx"
}
}
```

```sh
terraform apply \
-var 'region=us-west-1' \
--auto-approve
```


## live EKS cluster 구축하며 생긴 일

1. VPC 개수는 리전당 5개이다. 
2. IGW 는 VPC와 1:1 매핑이다. (즉 마찬가지로 5개 한정)
3. 우리는 AWS CloudFormation과 Terraform, Terragrunt을 같이 사용하였다. 
4. AWS CloudFormation은 eks 클러스터 생성할 때 사용 -> Terraform에 넘겨야 할 인자들이 너무 많아서 그냥 CloudFormation을 사용했는데 형상 관리가 안돼서 불편하다고 한다.
5. Terraform은 포맷을 만들고, Terragrunt는 변수를 만들어 Terrafrom에서 만든 포맷에 변수를 넣어 사용한다. 
6. (dev, stag, live 모두 동일한 포맷으로 동작해야 하는데 MSK type 등 달라지는 것만 변수화 하는 것이다.)
7. Terraform의 tfstate는 S3로 관리하였다. apply한 결과를 원격으로 저장해서 이 값을 다음 작업에 사용할 수 있었다. 
8. 예를 들면 vpc를 nancy님 pc로 생성해도 해당 vpc 정보를 사용하는 MSK를 만들 때는 내 pc를 이용해도 문제 없는 것이다. (설정이 원격으로 저장되고 불러오기에)

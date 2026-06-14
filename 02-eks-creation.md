# EKS Setup

## EKS Clusters Creation

eksctl create cluster --name hub-cluster --region us-west-1
<img width="1422" height="617" alt="Screenshot 2026-06-14 162557" src="https://github.com/user-attachments/assets/b6c96e09-4226-48f2-ba9c-d74083e35896" />


eksctl create cluster --name spoke-cluster-1 --region us-west-1
<img width="1442" height="643" alt="Screenshot 2026-06-14 162614" src="https://github.com/user-attachments/assets/b01b68a8-5a45-42d2-9c74-1f082186288f" />


eksctl create cluster --name spoke-cluster-2 --region us-west-1
<img width="1418" height="654" alt="Screenshot 2026-06-14 162634" src="https://github.com/user-attachments/assets/10c06c02-d6bb-4b5b-8302-40514ca8c207" />


## EKS Clusters Deletion

eksctl delete cluster --name hub-cluster --region us-west-1
<img width="1155" height="346" alt="Screenshot 2026-06-14 163728" src="https://github.com/user-attachments/assets/eb21d71c-31f3-429b-b0c4-0c7fed09033c" />


eksctl delete cluster --name spoke-cluster-1 --region us-west-1
<img width="1012" height="354" alt="Screenshot 2026-06-14 163742" src="https://github.com/user-attachments/assets/139b44ca-d99b-4b16-bf0e-7a1e14fdba31" />


eksctl delete cluster --name spoke-cluster-2 --region us-west-1
<img width="1056" height="362" alt="Screenshot 2026-06-14 163753" src="https://github.com/user-attachments/assets/a699378c-8630-47a9-8870-1d1250306fe1" />

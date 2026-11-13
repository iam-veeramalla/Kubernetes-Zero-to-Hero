"Your Pod receives AccessDenied: Not authorized to perform sts:AssumeRoleWithWebIdentity. How do you troubleshoot?"

A strong answer is:

"This error indicates that AWS STS rejected the request to assume the IAM Role.
I would first verify that the Pod is using the correct ServiceAccount, confirm the ServiceAccount has the correct IAM Role annotation, 
check that the IAM Role's trust policy matches the ServiceAccount and namespace (system:serviceaccount:<namespace>:<serviceaccount>), verify that the EKS cluster's OIDC provider matches the trust policy, and ensure the OIDC provider is associated with the cluster. 
If STS succeeds but S3 still returns AccessDenied, I would then review the IAM policy attached to the role."

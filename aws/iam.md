## AWS IAM

Instance profiles are used to define EC2 instances. These profiles can link instances with IAM Roles which define what operations can be done by the instances. The IAM Roles can have Trust Policies to allow EC2 instances to “assume” these roles, which is the action that attaches an IAM role to an instance profile.
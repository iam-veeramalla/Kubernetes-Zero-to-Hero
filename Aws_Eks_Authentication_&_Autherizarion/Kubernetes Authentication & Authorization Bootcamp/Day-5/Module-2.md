Module 2: RBAC Has No Explicit Deny
====================================
This is one of the most common interview questions.

Many IAM systems have:

Allow

Deny

Kubernetes RBAC does not.

It only understands:

Allow

or

Not Allowed

Example:

Developer Role

get pods
list pods

Another Role

delete pods

Effective result:

get

list

delete

There is no rule like:

deny delete pods

because RBAC doesn't support deny rules.

If a permission is granted anywhere, the user has it.

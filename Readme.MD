#### In this repo we will understand about K8s RBAC
        Role
        RoleBinding
        ServiceAccount and 
        ClusterRole

#### Understanding RBAC

    RBAC is basically about Authentication and Authorization

    Nouns --> names
    Verbs --> what action we do

    Services -> EC2, R53, VPC, etc
    Verbs --> createInstance, GetInstance, UpdateInstance, DeleteInstance

    Similarly in terms of k8s, we have the actions like createPod, ReadPod, UpdatePod, DeletePod...etc

    In general, we may have the access criteria based on our Job role or Job level
    example:-
        Roles	   Permissions
        Trainee	   read all resources in roboshop
        Junior	   create Pod
        Senior	   Create Update
        Team Lead  Delete access
    So, for this we create roles and attach them to the specific rolebinding i.e., (Which role is binded to which user)

#### Integrating IAM and RBAC in K8s

    In general, IAM and RBAC in k8s are two different services, we have to integrate them first

        -- create an IAM user
        -- He need to list his cluster
        -- we need to integrate IAM user with k8 Role

    User creation Steps
        IAM >>> create user
        Policies >>> create a policy >>> select a service(EKS) >>> Read (Describe cluster) >>> Add ARNs (we need to give our region name and clustername so that it forms the ARN) >>> Now give the policy name and create a policy
        IAM >>> user >>> attach policy >>> search for the policy we created and attach it to the user we created
    
    For this user we have to create a Role and attach it to Rolebinding

    #### Yaml files for Creating a Role for a user(trainee) and we attach this role to a rolebinding

        apiVersion: rbac.authorization.k8s.io/v1
        kind: Role
        metadata:
            namespace: roboshop
            name: roboshop-trainee
        rules:
        - apiGroups: [""] # "" indicates the core API group(deployments, services, etc...)
          resources: ["pods"] # nouns
          verbs: ["get", "watch", "list"] # actions
        -----
        apiVersion: rbac.authorization.k8s.io/v1
        # This role binding allows "jane" to read pods in the "default" namespace.
        # You need to already have a Role named "pod-reader" in that namespace.
        kind: RoleBinding
        metadata:
            name: suresh
            namespace: roboshop
        subjects:
        # You can specify more than one "subject"
        - kind: User
          name: suresh # "name" is case sensitive
          apiGroup: rbac.authorization.k8s.io
        roleRef:
            # "roleRef" specifies the binding to a Role / ClusterRole
            kind: Role #this must be Role or ClusterRole
            name:  roboshop-lead # this must match the name of the Role or ClusterRole you wish to bind to
            apiGroup: rbac.authorization.k8s.io

    Above is the role created for a user(trainee) and we are attaching this role to rolebinding.
    Rolebinding will have the info about the role and the concerned user which we created in AWS IAM

    This user is like an EKS Admin or something related to EKS team.
    And he has to login to the newly created EC2 instance and do a regular AWS configure i.e., using secretkey and access keys.
    Create a the secretkey and accesskey for this user and he should loginto the newly created ec2 via aws console using the credentials

    

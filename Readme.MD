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

    Now we have to pull the code code related to the RBAC into our regular worker node from where we are doing all the kubectl operations and create the role and rolebinding also make sure we create a namespace before this...

    To integrate the AWS IAM user with the k8s, we have some configmap in the kubesystem namespace and AWS has provided a document as well -- https://docs.aws.amazon.com/eks/latest/userguide/auth-configmap.html

    Below is the Configmap which we have in kubesystem namespace 

    apiVersion: v1
    data:
        mapRoles: |
            - rolearn: arn:aws:iam::315069654700:role/eksctl-roboshop-dev-nodegroup-robo-NodeInstanceRole-TtRghdCKdBjV
            groups:
                - system:bootstrappers
                - system:nodes
            username: system:node:{{EC2PrivateDNSName}}
        mapUsers: |
            - groups:
              - roboshop-trainee  # role which we created for the user
            userarn: arn:aws:iam::315069654700:user/suresh   # arn of the user which we created
            username: suresh # username which we created
    kind: ConfigMap
    metadata:
        creationTimestamp: "2025-08-06T01:47:38Z"
        name: aws-auth
        namespace: kube-system
        uid: 7c47ebd8-d158-4e4e-b0e3-dce2584f27a6

    Now, if we apply this aws-auth.yaml file, this user will be added to our kebesystem and he can access the cluster with the given permissions

    If we check the configmap after applying, we can see the extra annotations

    ### NOW USER NEEDS TO LOGINTO THE NEWLY CREATED EC2 MACHINE

        make sure we have kubectl on this...

        curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.5/2025-09-19/bin/linux/amd64/kubectl   >>> this command to install kubectl

        chmod +x ./kubectl   >>> providing execute permissions

        sudo cp kubectl /usr/local/bin/kubectl   >>> copying kubectl to aother folder

        Done, now user can have access

        aws sts get-caller-identity   >>> command to see who has loggedin
            this command displays the user who has logged in and his details like ARN and the AWS accountID
        aws eks update-kubeconfig --region us-east-1 --name roboshop-dev  >>> Connect kubectl to an EKS cluster by creating a kubeconfig file i.e., to have the EKS access

        https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html  >>> refer this do for the above commands and as well as to understand how the authentication happens in an EKS cluster 

        Now, when we hit kubectl get pods >>> this will try to get the pods in a default namespace and as a result we get error on the screen because we have given access to the roboshop namespace

        Authentication is successful and Authorization is failure. To over come this please mention the namespace in the command

        If u notice the role yaml file, we are maintaining some rules, which include access to the resource and what action the user can take on thta resource.
        Always make sure the rules are mentioned with in the role only...
        Understand that for every role we create we should maintain a specific rolebinding for it to bind the user(role)

        But, in our case, we are giving the user name itself in the rolebinding yaml file

        #### If the hirearcy of this user in the organisation is increasing, we can give access to all the resources and take the necessary actions accourding to the rules we mentioned in this role

        ### He can get the info about the pvc as this is a namespace level object and if this user wants to see the object at the cluster level, we have something called clusterRole

        Below are the yamls for ClusterRole and ClusterRolebinding

        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRole
        metadata:
          name: roboshop-cluster
        rules:
        - apiGroups: [""]
          resources: ["persistentvolumes"]
          verbs: ["get", "list", "watch"]
        ---
        apiVersion: rbac.authorization.k8s.io/v1
        # This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
        kind: ClusterRoleBinding
        metadata:
            name: roboshop-cluster
        subjects:
        - kind: User
          name: suresh # Name is case sensitive
          apiGroup: rbac.authorization.k8s.io
        roleRef:
          kind: ClusterRole
          name: roboshop-cluster
          apiGroup: rbac.authorization.k8s.io

    Apply these two files via the workernode and see for the commands related to pv, now the user can see the cluster level objects like PV

#### Service Account

    It is non human user that pod uses to run. by default when we create namespace a serviceaccount with name is default is created.

    Default service account will not have much permissions and these are created for thr pod i.e., if  pod want to communicate with other services in the cluster or if it want to fetch any details like passwords from AWS, it uses SA and communicates with other services

    #### Steps to integrate a ServiceAccount in a Cluster

    We need to have an OIDC provider as we are going to interact with AWS for secrets

    1. Create OIDC provider

    REGION_CODE=us-east-1
    CLUSTER_NAME=roboshop-dev
    ACC_ID=315069654700

    eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve

    2. create a secret with the name RoboShopMySQLSecretReader and create a policy for the secretmanager(GetSecretValue) and we can mention the ARN of the secret which we created.

    arn:aws:iam::315069654700:policy/RoboShopMySQLSecretReader   >>> ARN of Policy

    3. creating service account

    eksctl create iamserviceaccount \
    --cluster=$CLUSTER_NAME \
    --namespace=roboshop \
    --name=roboshop-mysql-secret-reader \
    --attach-policy-arn=arn:aws:iam::315069654700:policy/RoboShopMySQLSecretReader \
    --override-existing-serviceaccounts \
    --region $REGION_CODE \
    --approve

    this command creates IAM role and SA and integrates them

    In the above command make sure we have the arn of the policy and the secretname and namespace as roboshop and please change the region and the account id accordingly

    When we are running the aove command to create the SA, by default AWS is creating a role and attaching it to our SA. We can check this in our SA yaml file
    kbectl get sa roboshop-mysql-secret-reader -n roboshop -o yaml
    we can see a role is created and attached to this SA

    Now, as we have service account in place, we should create a role and rolebinding for this SA

    Below are the yamls for role, rolebinding and pod

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
        namespace: roboshop
        name: sa-reader
    rules:
    - apiGroups: ["*"] # "" indicates the core API group
      resources: ["*"] # nouns
      verbs: ["get", "watch", "list"] # actions
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    # This role binding allows "jane" to read pods in the "default" namespace.
    # You need to already have a Role named "pod-reader" in that namespace.
    kind: RoleBinding
    metadata:
      name: sa-reader
      namespace: roboshop
    subjects:
    # You can specify more than one "subject"
    - kind: ServiceAccount
      name: roboshop-mysql-secret-reader # "name" is case sensitive
      namespace: roboshop
    roleRef:
      # "roleRef" specifies the binding to a Role / ClusterRole
      kind: Role #this must be Role or ClusterRole
      name:  sa-reader # this must match the name of the Role or ClusterRole you wish to bind to
      apiGroup: rbac.authorization.k8s.io

    https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/get-secret-value.html  >>> use his document To retrieve the encrypted secret value of a secret

    aws secretsmanager get-secret-value \
    --secret-id roboshop/mysql/password
    this is the command from the above document to retrive the secret
    roboshop/mysql/password >>> this is the secret we created in the AWS
    Now the above command will display the value in our secret.

    And now, we should be getting this secret via a pod by using SA i.e., we should(pod) be able to access the secrets in AWS by using the SA created for it 

    Now we should create a pod for this SA using the aws-cli image

    apiVersion: v1
    kind: Pod
    metadata:
      name: aws-cli
      namespace: roboshop
      labels:
        purpose: sa
    spec:
      serviceAccount: roboshop-mysql-secret-reader
      containers:
      - name: awscli
      image: amazon/aws-cli
      command: ["sleep","10000"]

      after executing this command, get inside the pod using kubectl exec -it asw-cli -n roboshop and if we run the command 
      aws secretsmanager get-secret-value \
      --secret-id roboshop/mysql/password  
      we can see that the pod is fetching the secret from AWS and displaying to us using a SA.

    As we are having ASW CLI, we are able to look at fetch and look at our secret value from the workernode, but for pods in our applications we dont use aws-cli image and as a result we have something called init containers

#### Final Understanding on ServiceAccount

    As a humans if we want to access the resources in k8, we create an IAM user, attach policy to the user and create role and rolebinding and attach it to the user and finally when we created all the resources like role, rolebinding, we are able to access and take the necessary action on the k8 resources using that IAM user.

    If a pod want to communicate with the resources outside the cluster, it should have the ServiceAccount

    https://youtu.be/hYQnleGXnSQ?si=63-SX1UMKqYPM7KI  >>>> watch this video to understand the topics like role, rolebinding and the serviceaccount
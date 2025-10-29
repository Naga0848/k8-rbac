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

    For this we have to create a Role and Rolebinding(this has to be attached an IAM user which we created)  

    
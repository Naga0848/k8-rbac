#### In this repo we will understand about K8s RBAC
        Role
        RoleBinding
        ServiceAccount and 
        ClusterRole

#### understanding RBAC

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

    
        
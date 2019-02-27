# jenkins-cd


1.  Create a working Kubernetes Cluster (AWS/GCP/Azure/Onprem)
 
 gcloud container clusters create jenkins-cd1 --num-nodes 2 --machine-type n1-standard-2 --scopes "https://www.googleapis.com/auth/projecthosting,cloud-platform"
 gcloud container clusters get-credentials jenkins-cd1
 
 kops create cluster kubernetesfederatedcluster.com
 
2.  Clone the sample app 

3.  Configure your favorite version control system - git / GCP container-registry / AWS container registry

4.  Install helm 

5.  Configure tiller '

6.  kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)  --- only for GCP users/ AWS users couple IAM with EKS 

7.  kubectl create serviceaccount tiller --namespace kube-system

    kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
    
    helm init --service-account=tiller
    
    helm update
    
8.  Install Jenkins as a container

    helm install -n cd stable/jenkins -f jenkins/values.yaml --version 0.16.6 --wait
    
9.    printf $(kubectl get secret --namespace default cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
      
    The above command prints password for jenkins 

10. kubectl edit svc cd-jenkins

11. Change ClusterIP to NodePort or LoadBalancer

12. Use externalIP:NodePort to launch jenkins - http://35.202.22.239:30060/login

13. Login using password and username=admin

14. Manually deploy SampleAPP ---- This is only for our understanding of canary deployment. This step is not required

15. kubectl create ns production
    
    kubectl --namespace=production apply -f k8s/production
    
    kubectl --namespace=production apply -f k8s/canary
    
    kubectl --namespace=production apply -f k8s/services
    
    kubectl --namespace=production scale deployment gceme-frontend-production --replicas=4

16. On a separate tab - 

    export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}"  --namespace=production services gceme-frontend)
    
    while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1;  done

    Output is 1.0.0
    
17. Create your own repo - you can use github / cloud-repositories etc

18. git init 

    git config credential.helper gcloud.sh --- for GCP
    
    Create external repo
    
    gcloud source repos create gceme2
    
     git remote add origin https://source.developers.google.com/p/advanced-kubernetes-harshal/r/gceme2

19. git config --global user.email "sharma.hrsh6@gmail.com"

20. git config --global user.name "Harshal" 

21. git add .

22. git commit -m "Initial commit"

23. git push origin master

23. Add Credentials to Jenkins

24. On Jenkins Credentials - 
    
        1.  Click Global
        
        2.  Add Credentials
        
        3.  Add all credentials for registry 
        
25. Create a multipipeline job
        
        1.  Branch Source - your repo
        
            https://source.developers.google.com/p/advanced-kubernetes-harshal/r/gceme2
            
        2.  Credentials - your credentials for repository
        
        3.  Scan Multibranch Pipeline Triggers - 1 min 
        
26. git checkout -b canary

27. Edit the below files - 

    Jenkinsfile  -- Add your repository details  : def project = 'advanced-kubernetes-harshal'

    html.go -- Change text orange to blue
    
    main.go  - Change version from 1.0.0 to 2.0.0
    
28. git add Jenkinsfile html.go main.go

29. git commit -m "Version 2"

30. git push origin canary

31. Scan the Pipeline 

32. You should start getting new version on the screen

33. git checkout master

    git merge canary
    
    git push origin master





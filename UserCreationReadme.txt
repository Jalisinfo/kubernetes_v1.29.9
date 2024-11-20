Here are all the steps to follow:- UPPERCASE are the commands

Create a user in the server if doesn't exist.  
USERADD -c "jmc" -m jmc  ***-> where jmc is the username and with -m jmc being the group name

Once user is created in the server, let's create private key for the user by using openssl
OPENSSL genrsa -out jmc.pem **-> this will create RSA private key for jmc user. If you want can provide 4096 as the end of the command to have the key generated longer in bit.

Let's now generate a CSR request for the jmc user by using the private key created in step-7
OPENSSL req -new -key jmc.pem -out jmc.csr -subj "/CN=JMC/O=DevOps"


####APPROACH -1 (for getting the CERT signed and approved VIA Kubernetes cluster):- ####

We then need to convert this newly created CSR which base64 encoded to a DER format by using following command:
CAT jmc.csr | base64 | tr -d "\n"
===
### Next, we need to create a custom resource file for kubenestes signing request:-

look for the YAML file call - jmc-user-cert-csr-from-kub.yaml

Once creatd the CSR request in kubernetes, it will show as below:-

jmc-user-csr   24s   kubernetes.io/kube-apiserver-client   kubernetes-admin   19h                 Pending

Next Approve the cert request:-

kubectl certificate approve jmc-user-csr
certificatesigningrequest.certificates.k8s.io/jmc-user-csr approved

Once Approved , it will show ike this:-
jmc-user-csr   2m43s   kubernetes.io/kube-apiserver-client   kubernetes-admin   19h                 Approved,Issued

Next, is to extract the certificate from the appoved CSR:-
kubectl get csr jmc-user-csr -o jsonpath="{.status.certificate}" | base64 -d > /home/jmc/jmc.crt

####APPROACH -2 (Getting the CERT signed VIA Certificate Authority):- ####
Generate User Private key:-
1. openssl genrsa -out jmc.key 4096

2. Let's now generate a CSR request for the jmc user by using the private key created in step-1
openssl req -new -key jmc.key -out jmc.csr -subj "/CN=jmc/O=DevOps" 

Important notes:-
<- CN= jmc is the user O=DevOps is refering user jmc belongs to the DevOps group, which is then refers to an existing namespace called DevOps your kubernetes.

3. Singe the CSR request by the Cert Authority:-

openssl x509 -req -in jmc.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out jmc.crt -days 365

Important notes:-
<- make sure your ca.key and ca.crt is in the same directory as with jmc.key

4. Create a kubeconfig file for the user jmc:-
kubectl --kubeconfig jmc.kubeconfig config set-cluster kubernetes --server https:<YOUR CLUSTER IP>:6443 --certificate-authority=ca.crt

### ***if you want to share the ca.crt, user key and user crt for user to create the kubeconfig file then follow below:-**** And user, thi case jmc logs in and does perform steps: 5-6

5. Set/Add the jmc user to the kubeconfig file created in setp-4
kubectl --kubeconfig jmc.kubeconfig config set-credentials jmc --client-certificate /home/jmc/jmc.crt --client-key /home/jmc/jmc.key  

Important notes:-
<- need to provide full path of the user jmc crt and key is a must.

6. Set/Add the user context to the kubeconfig file created in setp-4
kubectl --kubeconfig jmc.kubeconfig config set-context jmc-kubernetes --cluster kubernetes --namespace default --user jmc

7. vi jmc.kubeconfig and edit current-context : jmc-kubernetes
Important notes:- 
current-context cannot leave as empty and must be the same a username

8. create a .kube DIR under the jmc user HOME DIR and copy the jmc.kubeconfig as config file under .kube DIR


###***if you want to create the kubeconfig file then share with user only the config file then follow below:-****

In this approach : the field names are gonna be: clinet-key-data instead of just clinet-key, client-certificate-date instead of just client-certificate, and certificate-authority-data instead of just certificate-authority - beacuse we are passing file not not the file reference.

copy your admin config file and start modifying the it with jmc user data:-

make sure encode the user key, crt, ca crt in base64:-

cat jmc3.crt | base64 -w0  <- here w0 removes the line breaks.

9. Create role, roleBinding for the user jmc to access the kubernetes resources/objects 

*** check the filename call - jmc-grant-permission.yaml













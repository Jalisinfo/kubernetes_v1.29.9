Here are all the steps to follow:- UPPERCASE are the commands

Create a user in the server if doesn't exist.  
USERADD -c "jmc" -m jmc  ***-> where jmc is the username and with -m jmc being the group name

Once user is created in the server, let's create private key for the user by using openssl
OPENSSL genrsa -out jmc.pem **-> this will create RSA private key for jmc user.

Let's now generate a CSR request for the jmc user by using the private key created in step-7
OPENSSL req -new -key jmc.pem -out jmc.csr -subj "/CN=JMC"

We then need to convert this newly created CSR which base64 encoded to a DER format by using following command:
CAT jmc.csr | base64 | tr -d "\n"
===
### Next, we need to create a custom resource file for kubenestes signing request:-

look for the YAML file call - jmc-user-cert-csr-from-kub.yaml



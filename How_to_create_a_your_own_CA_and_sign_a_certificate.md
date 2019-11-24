# How to create a your own CA and sign a certificate for your services
Two steps:

1. **Create CA artifacts**
2. **Create Service’s artifacts**

Here, Lets assume that

- ‘**new_ca**’ is the the CA (Common Name = ‘admin.new-ca.com’)
- ‘**test.frontline.com**’ is the service which we want to get a certificate installed in it ( Common Name = ‘**test.frontline.com**’)


> There is nothing to be worried about the **Common Name of the CA**, but service’s Common Name is essential between the end-to-end certificate creation process
# Create CA artifacts

This step consist of 3 tasks

> Better if you can go to a separate directory for better security practices 

## Task 1: Create CA Private Key

Here we are using RSA to generate the private key

    openssl genrsa -out ca_private.pem 2048
## Task 2: Extract the public key from private key

This task is not necessary to do. But, you may need the public key to verification

    openssl rsa -in ca_private.pem -outform PEM -pubout -out ca_public.pem
## Task 3: Create certificate for CA

Create a self signed root certificate for CA

    openssl req -x509 -new -nodes -key ca_private.pem -sha256 -days 1825 -out new_ca.pem

After executing this command you will be asking questions about the CA. You have to answer them appropriately

Here, **“new_ca.pem”** is the root certificate of the CA

# Create Service’s artifacts

We are now creating the **‘test.frontline.com’** service’s certificate signed by the CA. Here also we are following 4 tasks before creating the certificate

## Task 1: Create private key for ‘test.frontline.com’
    openssl genrsa -out test.frontline.com.key 2048
## Task 2: Create a CSR
    openssl req -new -key test.frontline.com.key -out test.frontline.com.csr

As earlier asked, after executing this command you will be asking questions about the CA. You have to answer them appropriately. Please make sure you have put **test.frontline.com** as the common name
****
## Task 3: Creating a config file 

create file called *******test.frontline.com.ext*** containing some configurations

    authorityKeyIdentifier=keyid,issuer
    basicConstraints=CA:FALSE
    keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
    subjectAltName = @alt_names
    
    [alt_names]
    DNS.1 = test.frontline.com
    DNS.2 = test.frontline.com.192.168.1.19.xip.io
## Task 4: Sign and create a certificate for the service using the CSR and EXT file
    openssl x509 -req -in test.frontline.com.csr -CA new_ca.pem -CAkey ca_private.key -CAcreateserial -out test.frontline.com.pem -days 1825 -sha256 -extfile test.frontline.com.ext


# Post Quantum signatures for TLS handshake.

## Creating the post quantum signatures keys and certificate for TLS hanshake using OpenSSL3 / Oqs-provider(Open Quantum Safe provider).


The mian motive of this project is to create the keys and certificate using post quantum signatures with the help of openssl3 and oqsprovider.

Note : As we used the nginx server or stunnel4 as a proxy server to make the DOT queries. Similarly for this project we tried to use the above certificate and key for TLS hankshake but the presently there is no support for handling the post qunatum signaturs by nginx and stunnle, so either we have to create our own setup which will support the post qunatum signatures or we need wait wait for future modification by devlopers of Oqs-providers.


Please follow the step by step process to cretate whole setup. 

### Step 1 : Installation of pre-requisites

For building, minimum requirements are a C compiler, git access and cmake. For Linux these commands can typically be installed by running for example

```
sudo apt install build-essential git cmake
```

### Step 2: Installing the OpenSSL3

The following shows an example for building and installing the latest/master branch of OpenSSL 3 in .local:

```
sudo git clone git://git.openssl.org/openssl.git
cd openssl
sudo ./config --prefix=$(echo $(pwd)/../.local) && sudo make && sudo make install_sw
cd ~
``` 

### Step 3: Installing liboqs

Example for building and installing liboqs in .local:

Note : Please replace all the path as per your system configuration, for an example , if you want path for your .local directory just do 
``` mlocate .local ```  this command will give you the exact path, copy the path and add it into the command and run it. 
```
sudo git clone https://github.com/open-quantum-safe/liboqs.git
cd liboqs
sudo apt install doxygen graphviz
sudo cmake -DOPENSSL_CRYPTO_LIBRARY=/home/openssl3/.local/lib64/libcrypto.so -DOPENSSL_ROOT_DIR=/home/openssl3/openssl -DCMAKE_INSTALL_PREFIX=$(pwd)/../.local -S . -B _build
sudo cmake --build _build
sudo cmake --install _build
```

### Step 4 : Building the provider 

oqsprovider using the local OpenSSL3 build as done above can be built for example via the following:

i. Getting the source to Build the provider 
```
sudo git clone https://github.com/open-quantum-safe/oqs-provider.git
cd oqs-provider
sudo cmake -DOPENSSL_ROOT_DIR=/home/openssl3/.local -DOPENSSL_LIBRARIES=/home/openssl3/.local/lib64 -DOPENSSL_INCLUDE_DIR=/home/openssl3/.local/include -Dliboqs_DIR=/home/openssl3/.local/lib/cmake/liboqs -S . -B _build
sudo cmake --build _build
```

ii. After bulding we need to execute below command to check whether it was build properly or not. 

 note : run the below command in the oqs-provider directory itself.

```
cd _build/test/
sudo ctest --parallel 5 --test-dir _build --rerun-failed --output-on-failure
```
We will get below output as a result.  

output : 

```
Test project /home/openssl3/oqs-provider/_build/test
    Start 5: oqs_endecode
    Start 1: oqs_signatures
    Start 4: oqs_tlssig
    Start 3: oqs_groups
    Start 2: oqs_kems
1/5 Test #2: oqs_kems .........................   Passed    0.38 sec
2/5 Test #3: oqs_groups .......................   Passed    0.56 sec
3/5 Test #4: oqs_tlssig .......................   Passed    2.75 sec
4/5 Test #1: oqs_signatures ...................   Passed    3.08 sec
5/5 Test #5: oqs_endecode .....................   Passed    5.49 sec

100% tests passed, 0 tests failed out of 5

Total Test time (real) =   5.49 sec
````

iii. Once testing done, run the below command to come out from ( _build/test/ ) path in oqs-provider directory and installing the provider. 

```
cd ../../
sudo cmake --install _build
```
iv. To make our custom openssl3 as default openssl.

``` sudo nano ~/.bashrc ```

and add the following in the bottom of the file :

```
export OPENSSL_ROOT_DIR=/home/openssl3/openssl
export OPENSSL_CONF=/etc/ssl/openssl.cnf
export OPENSSL_MODULES=/home/openssl3/oqs-provider/_build/lib
export PATH="/home/openssl3/.local/bin:$PATH"
export LD_LIBRARY_PATH=/home/openssl3/.local/lib64:$LD_LIBRARY_PATH
```
save the file and run the below: 

```
source ~/.bashrc
openssl verion
```
output : 

![image](https://github.com/MeherP2246/FIT-DNS-RESEARCH-/assets/134104519/9fde27d7-a9ec-4161-81e3-83a5e287cc50)



v. To enable oqsprovider by default in the system's openssl.cnf file, modify the "[provider_sect]" in the following way:

Goto your openssl.cnf file : 

``` sudo nano /etc/ssl/openssl.cnf ```

and add the below lines in the bottom of the file : 

```
[provider_sect]
default = default_sect
oqsprovider = oqsprovider_sect
[oqsprovider_sect]
activate = 1
```
save the file and run the below command to check the provider version information 

```
LD_LIBRARY_PATH=/home/openssl3/.local/lib64  /home/openssl3/.local/bin/openssl list -providers -verbose -provider-path /home/openssl3/oqs-provider/_build/lib -provider oqsprovider

openssl list -providers -verbose
```
The output :

![image](https://github.com/MeherP2246/FIT-DNS-RESEARCH-/assets/134104519/7060fe9d-d3a1-4287-bf99-d10c4365dd28)


vi. Also we can check the list of algorithms that suppoted by oqs-provider using below command : 

``` openssl list -all-algorithms -provider-path /home/openssl3/oqs-provider/_build/lib -provider oqsprovider ```

### Step 5 : Creating the keys and certificate using post quantum signatures 

 1. Falcon512

```
sudo LD_LIBRARY_PATH=/home/openssl3/.local/lib64  /home/openssl3/.local/bin/openssl req -x509 -new -newkey falcon512 -keyout qsc.key -out qsc.crt -nodes -subj "/CN=oqstest" -days 365 -config /home/openssl3/openssl/apps/openssl.cnf -provider-path /home/openssl3/oqs-provider/_build/lib -provider oqsprovider -provider default
```
2. p256_falcon512
```
sudo LD_LIBRARY_PATH=/home/openssl3/.local/lib64  /home/openssl3/.local/bin/openssl req -x509 -new -newkey p256_falcon512 -keyout qsc.key  -out qsc.crt -nodes -subj "/CN=oqstest" -days 365 -config /home/openssl3/openssl/apps/openssl.cnf -provider-path /home/openssl3/oqs-provider/_build/lib -provider oqsprovider -provider default
```
3. p256_dilithium2
```
sudo LD_LIBRARY_PATH=/home/openssl3/.local/lib64  /home/openssl3/.local/bin/openssl req -x509 -new -newkey p256_dilithium2 -keyout abc.key -out abc.crt -nodes -subj "/CN=oqstest" -days 365 -config /home/openssl3/openssl/apps/openssl.cnf -provider-path /home/openssl3/oqs-provider/_build/lib -provider oqsprovider -provider default
```

### Example of creation and verification of quantum-safe digital signatures using  Cryptographic Message Syntax (CMS)

1.Signing data

In order to generate signed data, a two-step process must be followed: The first step involves generating a certificate employing a Quantum-Safe Cryptography (QSC) algorithm. Subsequently, this certificate, along with its associated signature algorithm, is utilized in the second step to produce the signed data.

  Step 1: Create quantum-safe key pair and self-signed certificate:
```
sudo LD_LIBRARY_PATH=/home/openssl3/.local/lib64  /home/openssl3/.local/bin/openssl req -x509 -new -newkey p256_falcon512 -keyout qsc.key  -out qsc.crt -nodes -subj "/CN=oqstest" -days 365 -config /home/openssl3/openssl/apps/openssl.cnf -provider-path /home/openssl3/oqs-provider/_build/lib -provider oqsprovider -provider default
```
By changing the -newkey parameter algorithm name any of the supported quantum-safe or hybrid algorithms can be utilized instead of the sample algorithm p256_falcon512.

   Step 2: Sign data:

As the CMS (Cryptographic Message Syntax) standard requires the presence of a digest algorithm, while quantum-safe crypto does not, in difference to the QSC certificate creation command above, passing a message digest algorithm via the -md parameter is mandatory. 

Note: here input file will be your any file which you want to sign and verify using post qunatum signature  

```
sudo LD_LIBRARY_PATH=/home/openssl3/.local/lib64  /home/openssl3/.local/bin/openssl cms -in inputfile -sign -signer qsc.crt -inkey qsc.key -nodetach -outform pem -binary -out signedfile -md sha512 -provider-path /home/openssl3/oqs-provider/_build/lib  -provider default -provider oqsprovider
```
Data to be signed is to be contained in the file named inputfile. The resultant CMS (Cryptographic Message Syntax) output is contained in file signedfile. The QSC algorithm used is the same signature algorithm utilized for signing the certificate qsc.crt.

2. Verifying data

Continuing the example above, the following command verifies the CMS file signedfile and outputs the outputfile. Its contents should be identical to the original data in inputfile above.

```
sudo LD_LIBRARY_PATH=/home/openssl3/.local/lib64 /home/openssl3/.local/bin/openssl cms -verify -CAfile qsc.crt -inform pem -in signedfile -crlfeol -out outputfile -provider-path /home/openssl3/oqs-provider/_build/lib -provider oqsprovider -provider default
```




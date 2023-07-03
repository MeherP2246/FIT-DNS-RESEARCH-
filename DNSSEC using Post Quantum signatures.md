# DNSSEC using Post Quantum signatures Algorithms .


The main motive of this project is to create a setup using Post Quantum signatures Algorithms with the help of openssl3 /oqsprovider, which will help to raise the DNSSEC queries using post qunatum signatures.   

Note : This custom BIND9 which is used for DNSSEC with Post Quantum signatures Algorithms is currently not supporting. 
For the  more updates please follow the (https://github.com/Martyrshot/OQS-bind.git) repository as we have already raised a issue with developer of BIND9.
and as per the discussion, he said that he will soon update the BIND9 repo which will support the DNSSEC with Post Quantum signatures Algorithms. 

 
Please follow the step by step process to create whole setup. 
_____________________________________________________________________________________________________________________________________________________________________

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

### Step 5 : OQS-bind9 (open quantum safe signature- bind9) 

TO start the oqs-bind we have to clone the following repository, please follow the below commad: 
```
https://gitlab.isc.org/isc-projects/bind9.git
cd bind9
sudo apt-get install liburcu-dev
sudo apt-get install libuv1-dev
sudo apt-get install libnghttp2-dev
sudo autoreconf -fi
LD_RUN_PATH=/home/openssl3/.local/lib ./configure --with-openssl=/home/openssl3/.local
sudo make 
```



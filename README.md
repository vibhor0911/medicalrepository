# medicalrepository
#digital_healthcare_fabric
This is api server which interacts with healtcare chaincode installed on hyperledger fabric basic_network.

Refer below urls.  
https://github.com/hyperledger/fabric-samples  


### Steps to setup chaincode:
* Bring fabric basic_network up  
* Register chaincode from root folder  
  ```CORE_CHAINCODE_ID_NAME="healthcare:v0" npm start -- --peer.address localhost:7052```
* Login to cli  
  ```docker exec -it cli bash```
* Install chaincode  
  ```peer chaincode install -n healthcare -v v0 -l node --cafile /etc/hyperledger/configtx/ -p /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/digital_health_fabric/```
  ##### Note: I had copied this project to crypto-config(basic_network) and attached volume to cli container using docker-compose
  #####       volumes: 
  #####          - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
* instantiate chaincode  
  ```peer chaincode instantiate -o orderer.example.com:7050 -C mychannel -n healthcare -l node -v v0 -c '{"args":["init"]}'```
* Optional invoke using cli,
  ```peer chaincode invoke -o orderer.example.com:7050 -C mychannel -n healthcare -c '{"Args":["registerPatient","patient5","20"]}'```
  
### Steps to interact with digital healthcare chaincode using node sdk.
* Remove any files from hfc-key-store folder
* Register admin to ca  
  ```node enrollAdmin.js```
* Start server  
  ```node server.js```
* Test chaincode and api  
  ```npm run test```
  
### List of apis
* Register doctor  
  ``` POST /register  
  {
    "name":"doctor1",
    "email": "doctor1",
    "password": "doctor1",
    "role": "doctor"
  }
  ```
* Register Patient  
  ``` POST /register  
  {
    "name":"patient1",
    "email": "patient1",
    "password": "patient1",
    "age": "24",
    "role": "patient"
  }
  ```
* Login patient and doctor  
  POST /login
  ```{
    "email": "patient2",
    "password": "patient2"
  }
  ```
  same for doctor  
  ##### Note: This api returns authtoken is login success. we need to pass this token as "authtoken" in header to other apis.  
* Patient grant access to doctor (use patient authtoken)  
  ```POST /modifyAccess  
  {
    "type": "grant",
    "doctorEmail": "doctor2",
    "org":"org1"
  }
  ```
* fetch patient info for doctor after access (use doctor authtoken, useremail=<patient_email>)  
   ```GET /getPatientInfo  
   ```
   ##### Note: patient info successfully fetched if and only if doctor has been granted access by patient.
* fetch patient info for himself (use patient authtoken)  
  ```GET /getPatientInfo```
  ##### Note: a patient can't see other patient's info.
* fetch doctor info for himself (use doctor authtoken)  
  ```GET /getDoctorInfo```
  ##### Note: a doctor can't see other doctor's info.
* patient upload file, first user has encrypt a file with secret and upload that file to ipfs.  
  hash returned from ipfs after upload and secret shoul be provided in below api, this api will attach file info to patient  
  ```POST /addFile  
  {
    "fileName":"abc.png", 
    "fileType":"png", 
    "secret":<ipfs file encryption secret>, 
    "fileHash":<ipfs file hash>
  }
  ```
* patient retrieve file info (use patient authtoken, filehash=<ipfs file hash>)  
  ```GET /getFileSecret```
* doctor retrieve file info (use doctor authtoken, filehash=<ipfs file hash>, useremail=<patient_email>)  
  ```GET /getFileSecret```
  ##### Note: doctor can retrieve file secret only if he as access to patient.  

  


# digital_healthcare_ethereum
electronic health records on blockchain

##### Note: Demo url to work you need to have metamask extension installed in your browser and to buy fake tickets you need to have fake ethers from rinkeby faceut.
##### Optimized contract can be found in contracts/optimized_healthCare.sol

### Summary
Project stores patient records on blockchain(hybrid). Hybrid because files are not stored on blockchain, but access information is stored on blockchain. There will be two participants doctor and patient.  
- Doctor register by providing name.  
- Patient register by providing name and age.
- Patient uploads files and provides random nounce to encrypt the file, file will be uploaded to IPFS and secret is stored in ethereum.
- Patient provides access to particular doctor.
- Once doctor is given access by patient, he will be able to see patient's address in his home page.
- Doctor can get all files ipfs hash of patient and send request to node app for file view.
- Node app will fetch file from ipfs and get secret from blockchain, decrypt file and send it to doctor.

### Note
Code has been tested only with ganache, not with any testnet.

### Project setup
HTTP_PROVIDER = provider url ex: http://127.0.0.1:7545  
IPFS_HOST = currently infura (can be changed to local node as well)

**1. Start Ganache**  
&nbsp;&nbsp;&nbsp;Contract can be deployed to any network, In my case ganache.
Update CONTRACT_DEPLOYED_PORT in env, which can be found in build -> contracts -> HealthCare.json -> "networks".  

**2. Start react server**  
&nbsp;&nbsp;&nbsp;`npm run start`  
&nbsp;&nbsp;&nbsp; visit http://localhost:3000  

**3. Start node app**   
&nbsp;&nbsp;&nbsp;`npm run server`  

**4. Connect metamask to ganache and Import ganache accounts to metamask**  
&nbsp;&nbsp;&nbsp;ex: http://localhost:7545  

___High Level Use Case___  

![Alt text](readme_images/high_level.png?raw=true "high_level")  

___Sign In___  

&nbsp;&nbsp;&nbsp; User should sign challenge to login, after which jwt token will be issued  

![Alt text](readme_images/2nd.png?raw=true "sign_in")  

___Upload Files___  

&nbsp;&nbsp;&nbsp; Here we have two layer of security 
1. hash provided by ipfs(ie. files can be accessed only if file hash is known)
2. file uploaded to ipfs is encrypted by secret (however secret is not encrypted in ethereum, should be done in future)

![Alt text](readme_images/3rd.png?raw=true "upload_files")  

___Access Files___  

![Alt text](readme_images/4th.png?raw=true "access_files")  

### TODO  
- Test cases, Currently deployment and registation tests has been written.
- Encrypt file secret while saving on ethereum (can be encrypted or NuCypher etc)

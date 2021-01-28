# Hyperledger Fabric Kubernetes Development Environment
This project provides a Hyperledger Fabric environment to test and develop without thinking about certificates.

## Deploy nodes
```bash
git clone https://github.com/agustincharry/hlf_k8s_cryptogen.git
cd hlf_k8s_cryptogen
kubectl apply -f .
```

## Validate the status of pods. -> Desired state: Running
```bash
kubectl get pods
```

## Install chaincode dependencies. Requires a bash shell!
```bash
cd chaincode-go
GO111MODULE=on go mod vendor
cd ..
```

## Copy chaincode to fabric-tools Pod
```bash
kubectl cp chaincode-go fabric-tools:/
```

## Get a shell of fabric-tools
```bash
kubectl exec -it fabric-tools -- /bin/sh
```

## On the fabric-tools shell
```bash
# Setting env variables
. /scripts/configClient.sh peer0
cd /artifacts/

# Creating the channel
export CHANNEL_ID=mychannel
peer channel create -o $ORDERER_ADDRESS -c $CHANNEL_ID -f ./channel.tx --tls --cafile $ORDERER_CA --certfile $CORE_PEER_TLS_CLIENTCERT_FILE --clientauth --keyfile $CORE_PEER_TLS_CLIENTKEY_FILE
peer channel join -b mychannel.block
peer channel list

# Installing Chaincode
peer lifecycle chaincode package basic.tar.gz --path /chaincode-go --lang golang --label basic_1.0
peer lifecycle chaincode install basic.tar.gz
peer lifecycle chaincode queryinstalled

# Approving Chaincode ->> CHANGE CC_PACKAGE_ID value!!
export CC_PACKAGE_ID=basic_1.0:3f11e6bf3bd16a30a6b434e227c679df6593537c1a48cbf3f61b0b33ad83a4b5
peer lifecycle chaincode approveformyorg -o $ORDERER_ADDRESS --channelID $CHANNEL_ID --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile $ORDERER_CA --certfile $CORE_PEER_TLS_CLIENTCERT_FILE --clientauth --keyfile $CORE_PEER_TLS_CLIENTKEY_FILE
peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_ID --name basic --version 1.0 --sequence 1 --tls --cafile $ORDERER_CA --output json

# Commiting Chaincode
peer lifecycle chaincode commit -o $ORDERER_ADDRESS --channelID $CHANNEL_ID --name basic --version 1.0 --sequence 1 --tls --cafile $ORDERER_CA --peerAddresses $CORE_PEER_ADDRESS --tlsRootCertFiles $CORE_PEER_TLS_ROOTCERT_FILE --certfile $CORE_PEER_TLS_CLIENTCERT_FILE --clientauth --keyfile $CORE_PEER_TLS_CLIENTKEY_FILE
peer lifecycle chaincode querycommitted --channelID $CHANNEL_ID --name basic --cafile $ORDERER_CA

# Invoking Chaincode
peer chaincode invoke -o $ORDERER_ADDRESS --tls --cafile $ORDERER_CA -C $CHANNEL_ID -n basic --peerAddresses $CORE_PEER_ADDRESS --tlsRootCertFiles $CORE_PEER_TLS_ROOTCERT_FILE -c '{"function":"InitLedger","Args":[]}' --certfile $CORE_PEER_TLS_CLIENTCERT_FILE --clientauth --keyfile $CORE_PEER_TLS_CLIENTKEY_FILE
peer chaincode query -C $CHANNEL_ID -n basic -c '{"Args":["GetAllAssets"]}'
```

## Delete all
```bash
kubectl delete -f .
```

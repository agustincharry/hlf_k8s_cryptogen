# Hyperledger Fabric Kubernetes

## Deploy nodes
```bash
cd hlf_k8s_cryptogen
kubectl apply -f .
```

## Install chaincode dependencies
```bash
cd chaincode-go
GO111MODULE=on go mod vendor
cd ..
```

## Copy chaincode to fabric-tools Pod
```bash
kubectl cp chaincode-go fabric-tools:/
```

## Getting a shell of fabric-tools
```bash
kubectl exec -it fabric-tools -- /bin/sh
```

## On the fabric-tools shell
```bash
# Setting env variables
. /scripts/peer0.sh
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

# Approving Chaincode - CHANGE CC_PACKAGE_ID!!
export CC_PACKAGE_ID=basic_1.0:6f3562abd7c619e2d242821e5415372eaec2d7a1812a0bd60af589e21a3d7d72
peer lifecycle chaincode approveformyorg -o $ORDERER_ADDRESS --channelID $CHANNEL_ID --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile $ORDERER_CA --certfile $CORE_PEER_TLS_CLIENTCERT_FILE --clientauth --keyfile $CORE_PEER_TLS_CLIENTKEY_FILE
peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_ID --name basic --version 1.0 --sequence 1 --tls --cafile $ORDERER_CA --output json

# Commiting Chaincode
peer lifecycle chaincode commit -o $ORDERER_ADDRESS --channelID $CHANNEL_ID --name basic --version 1.0 --sequence 1 --tls --cafile $ORDERER_CA --peerAddresses $CORE_PEER_ADDRESS --tlsRootCertFiles $CORE_PEER_TLS_ROOTCERT_FILE --certfile $CORE_PEER_TLS_CLIENTCERT_FILE --clientauth --keyfile $CORE_PEER_TLS_CLIENTKEY_FILE
peer lifecycle chaincode querycommitted --channelID $CHANNEL_ID --name basic --cafile $ORDERER_CA

# Invoking Chaincode
peer chaincode invoke -o $ORDERER_ADDRESS --tls --cafile $ORDERER_CA -C $CHANNEL_ID -n basic --peerAddresses $CORE_PEER_ADDRESS --tlsRootCertFiles $CORE_PEER_TLS_ROOTCERT_FILE -c '{"function":"InitLedger","Args":[]}' --certfile $CORE_PEER_TLS_CLIENTCERT_FILE --clientauth --keyfile $CORE_PEER_TLS_CLIENTKEY_FILE
peer chaincode query -C $CHANNEL_ID -n basic -c '{"Args":["GetAllAssets"]}'
```
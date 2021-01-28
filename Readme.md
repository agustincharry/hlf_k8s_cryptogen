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
. /scripts/peer0.sh
cd /artifacts/
# Creating the channel
export CHANNEL_ID=mychannel
peer channel create -o $ORDERER_ADDRESS -c $CHANNEL_ID -f ./channel.tx --tls --cafile $ORDERER_CA --certfile $CORE_PEER_TLS_CLIENTCERT_FILE --clientauth --keyfile $CORE_PEER_TLS_CLIENTKEY_FILE
peer channel join -b mychannel.block
peer channel list
```
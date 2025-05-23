UPDATE TO VERSION 0.13.0

#STOP NODE
```	sudo docker stop tanssi
	sudo docker rm tanssi
```

#PULL new docker images
```
docker pull moondancelabs/dancelight-chain:4
```

#RUN TANSSI NEW VERSION
```
export NODENAME=NODENAME
export VPSIP="wget -qO- eth0.me"
docker run --name tanssi --network="host" -v "/var/lib/tanssi-data:/data" \
-u $(id -u ${USER}):$(id -g ${USER}) \
--entrypoint "/chain-network/tanssi-relay" \
moondancelabs/dancelight-chain:4 \
--chain=dancelight \
--base-path=/data/ \
--unsafe-force-node-key-generation \
--database=paritydb \
--name=$NODENAME \
--listen-addr=/ip4/0.0.0.0/tcp/30333 \
--public-addr=/ip4/$VPSIP/tcp/30333 \
--state-pruning=archive \
--blocks-pruning=archive \
--rpc-cors=all \
--rpc-methods=unsafe \
--unsafe-rpc-external \
--rpc-max-connections=100 \
--telemetry-url='wss://telemetry.polkadot.io/submit/ 0' \
--validator
docker update --restart=unless-stopped tanssi
```

#Generate new session key

```curl http://127.0.0.1:9944 -H \
"Content-Type:application/json;charset=utf-8" -d \
  '{
    "jsonrpc":"2.0",
    "id":1,
    "method":"author_rotateKeys",
    "params": []
  }'
```
#MAP New session key and Finish!

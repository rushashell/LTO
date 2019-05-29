# Creating a LTO Node in a Azure Container Instance with CLI

Topics:
* [General](readme.md)
* [LTO node](lto-node-container-instance.md)
* [LTO node with public facing API](lto-node-container-instance.md)

# LTO node with public facing API

Welcome at this guide for setting up a LTO node within Azure. This is a quick-setup of a secure LTO node by using the Azure CLI and only executing a few commands!  

## Choose a deployment region
Because we are creating a node that will communicate over the internet, it is smart to think about the location we will be using. The further the node is away from other nodes, the longer it will take to sync and communicate (latency). To get a list of regions you can choose within your Azure tenant, execute the following command:

```
az account list-locations -o table --query "[].name"
```

Make a choice from the list that was returned and use it within our variable *CONTAINER_LOCATION*.

## Creating a Resource Group (Optional)

You always need a Resource Group to deploy a Container Instance in. If there is already an existing Resource Group, you can use that one. Be aware of the location that was set. 

In this step we assume we create a new Resource Group. I always use variables to keep the code clean and readable. Please setup your own variable values to comply with your needs: 

```
CONTAINER_RESOURCE_GROUP="lto-network-rg"
CONTAINER_LOCATION="westeurope"

# The actual command:
az group create --name $CONTAINER_RESOURCE_GROUP --location $CONTAINER_LOCATION
```

This should be finished quickly. You can now proceed to the next step.

## Create the Container
The container we will be creating is called an Azure Container Instance. Azure will take care of the fact it should be running always. 

Update the variables with values that comply with your needs. We will create the container by using the following commands:

```
CONTAINER_RESOURCE_GROUP="lto-network-rg"
CONTAINER_NAME="lto-network-node"

# Choose the network you want to connect your node to. The options are: MAINNET and TESTNET (default is MAINNET).
CONTAINER_NETWORK="MAINNET"

# Node name used in the handshake when connecting to other nodes
CONTAINER_NODE_NAME="YOUR NAME"

# Plain text seed for node wallet. Container converts it to base58.
CONTAINER_WALLET_SEED="SEED1 SEED2 SEED3"

# Password for wallet file.
CONTAINER_WALLET_PASSWORD="PASSWORD"

# Node logging level, available values: OFF, ERROR, WARN, INFO, DEBUG, TRACE.
CONTAINER_LOG_LEVEL="INFO"

# ApiKey used for the rest api authentication. 
CONTAINER_REST_API_KEY="CREATE YOUR OWN API KEY"

# The actual command:
az container create --resource-group $CONTAINER_RESOURCE_GROUP --name $CONTAINER_NAME \
 --image legalthings/public-node \
 --cpu 1 \
 --memory 2 \
 --os-type linux \
 --ip-address public \
 --ports 6868 6869 \
 --restart-policy Always \
 --environment-variables "LTO_NODE_NAME"="$CONTAINER_NODE_NAME" "LTO_ENABLE_REST_API"="TRUE" "LTO_LOG_LEVEL"="$CONTAINER_LOG_LEVEL" "LTO_NETWORK"="$CONTAINER_NETWORK" \
 --secure-environment-variables "LTO_WALLET_SEED"="$CONTAINER_WALLET_SEED" "LTO_WALLET_PASSWORD"="$CONTAINER_WALLET_PASSWORD" "LTO_API_KEY"="$CONTAINER_REST_API_KEY"
```

And that's it! You have a running LTO node that also has an API public available. Of course there are ways to check if it is indeed running, so let's check that out! 

## Check status of our node
We have a few ways to check the status of our (hopefully) running node. The easiest way is to check if the container is still running by using the following command:

```
az container show --resource-group lto-network-rg --name lto-network-node -o table
``` 
It should say Status:Running. If something is wrong, you can always check the logs:

```
az container logs --resource-group lto-network-rg --name lto-network-node --follow
```

Happy staking! 
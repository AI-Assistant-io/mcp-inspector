# Deployment setup
## Enable System-Assigned Managed Identity on the Container App
az containerapp identity assign --system-assigned --name mcp-inspector --resource-group aia-uks-rg-container-environment

## Get the Managed Identity's Principal ID  - STEP 2
az containerapp identity show --name mcp-inspector --resource-group aia-uks-rg-container-environment --query principalId --output tsv

## Get Your Subscription ID  - STEP 3
az account show --query id --output tsv

## Assign the AcrPull Role to the Identity
az role assignment create --assignee <principalId-from-step-2> --role AcrPull --scope /subscriptions/<subscriptionId-from-step-3>/resourceGroups/aia-uks-rg-container-environment/providers/Microsoft.ContainerRegistry/registries/ca72f2f56ae9acr

## Verify the Image Exists in ACR (to rule out push failures)
az acr repository list --name ca72f2f56ae9acr --output table

## Redeploy or Restart the Revision:
az containerapp update --name mcp-inspector --resource-group aia-uks-rg-container-environment

## Monitor logs
az containerapp logs show --name mcp-inspector --resource-group aia-uks-rg-container-environment --follow


# Deploy
## Build

docker build -f Dockerfile.local -t mcp-inspector:latest .

## Local docker deploy  
docker run --name mcp-inspector -p 6274:6274 -p 6277:6277 -d mcp-inspector:latest
docker run --name mcp-inspector -p 6274:6277 -d mcp-inspector:latest    

## local docker remove
docker rm -f mcp-inspector  

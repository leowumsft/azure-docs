---
title: Best practices in Azure Container Registry
description: Learn how to use your Azure container registry effectively by following these best practices.
services: container-registry
author: mmacy
manager: timlt

ms.service: container-registry
ms.topic: quickstart
ms.date: 12/20/2017
ms.author: marsma
---

# Best practices for Azure Container Registry

By following these best practices, you can help maximize the performance and cost-effective use of your private Docker registry in Azure.

## Network-close deployment

Create your container registry in the same Azure region in which you deploy containers. Placing your registry in a region that is network-close to your container hosts can help lower both latency and cost.

Network-close deployment is one of the primary reasons for using a private container registry. Docker images have an efficient [layering construct](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/) that allows for incremental deployments. However, new nodes need to pull all layers required for a given image. This initial `docker pull` can quickly add up to multiple gigabytes. Having a private registry close to your deployment minimizes the network latency.
Additionally, all public clouds, Azure included, implement network egress fees. Pulling images from one datacenter to another adds network egress fees, in addition to the latency.

## Geo-replicate multi-region deployments

Use Azure Container Registry's [geo-replication](container-registry-geo-replication.md) feature if you're deploying containers to multiple regions. Whether you're serving global customers from local data centers or your development team is in different locations, you can simplify registry management and minimize latency by geo-replicating your registry. Currently in preview, this feature is available with [Premium](container-registry-skus.md) registries.

To learn how to use geo-replication, see the three-part tutorial, [Geo-replication in Azure Container Registry](container-registry-tutorial-prepare-registry.md).

## Repository namespaces

By leveraging repository namespaces, you can allow sharing a single registry across multiple groups within your organization. Registries can be shared across deployments and teams. Azure Container Registry supports nested namespaces, enabling group isolation.

For example, consider the following container image tags. Images that are used corporate-wide, like `aspnetcore`, are placed in the root namespace, while container images owned by the Production and Marketing groups each use their own namespaces.

```
contoso.azurecr.io/aspnetcore:2.0
contoso.azurecr.io/products/widget/web:1
contoso.azurecr.io/products/bettermousetrap/refundapi:12.3
contoso.azurecr.io/marketing/2017-fall/concertpromotions/campaign:218.42
```

## Dedicated resource group

Because container registries are resources that are used across multiple container hosts, a registry should reside its own resource group.

Although you might experiment with a specific host type, such as Azure Container Instances, you'll likely want to delete the container instance when you're done. However, you might also want to keep the collection of images you pushed to Azure Container Registry. By placing your registry in its own resource group, you minimize the risk of accidentally deleting the collection of images in the registry when you delete the container instance resource group.

## Authentication

When authenticating with an Azure container registry, there are two primary scenarios: individual authentication, and service (or "headless") authentication. The following table provides a brief overview of these scenarios, and the recommended method of authentication for each.

| Type | Example scenario | Recommended method |
|---|---|---|
| Individual identity | A developer pulling images to or pushing images from their development machine. | [az acr login](/cli/azure/acr?view=azure-cli-latest#az_acr_login) |
| Headless/service identity | Build and deployment pipelines where the user isn't directly involved. | [Service principal](container-registry-authentication.md#service-principal) |

For in-depth information about Azure Container Registry authentication, see [Authenticate with an Azure container registry](container-registry-authentication.md).

## Manage registry size

The storage constraints of each [container registry SKU][container-registry-skus] are intended to align with a typical scenario: **Basic** for getting started, **Standard** for the majority of production applications, and **Premium** for hyper-scale performance and [geo-replication][container-registry-geo-replication]. Throughout the life of your registry, you should manage its size by periodically deleting unused content.

You can find the current usage of a registry in the container registry **Overview** in the Azure portal:

![Registry usage information in the Azure portal][registry-overview-quotas]

You can manage the size of your registry by using the [Azure CLI][azure-cli] or the [Azure portal][azure-portal]. Only the managed SKUs (Basic, Standard, Premium) support repository and image deletion--you cannot delete repositories, images, or tags in a Classic registry.

### Delete in Azure CLI

Use the [az acr repository delete][az-acr-repository-delete] command to delete a repository, or content within a repository.

To delete a repository, including all tags and image layer data within the repository, specify only the repository name when you execute [az acr repository delete][az-acr-repository-delete]. In the following example, we delete the *myapplication* repository, and all tags and image layer data within the repository:

```azurecli
az acr repository delete --name myregistry --repository myapplication
```

You can also delete image data from a repository by using the `--tag` and `--manifest` arguments. For details on these arguments, see the [az acr repository delete command reference][az-acr-repository-delete].

### Delete in Azure portal

To delete a repository from a registry in the Azure portal, first navigate to your container registry. Then, under **SERVICES**, select **Repositories**, and right-click the repository you want to delete. Select **Delete** to delete the repository and the Docker images it contains.

![Delete a repository in the Azure portal][delete-repository-portal]

In a similar manner, you can also delete tags from a repository. Navigate to the repository, right-click on the tag you wish to delete under **TAGS**, and select **Delete**.

## Next steps

Azure Container Registry is available in several tiers, called SKUs, that each provide different capabilities. For details on the available SKUs, see [Azure Container Registry SKUs](container-registry-skus.md).

<!-- IMAGES -->
[delete-repository-portal]: ./media/container-registry-best-practices/delete-repository-portal.png
[registry-overview-quotas]: ./media/container-registry-best-practices/registry-overview-quotas.png

<!-- LINKS - Internal -->
[az-acr-repository-delete]: /cli/azure/acr/repository#az_acr_repository_delete
[azure-cli]: /cli/azure
[azure-portal]: https://portal.azure.com
[container-registry-geo-replication]: container-registry-geo-replication.md
[container-registry-skus]: container-registry-skus.md

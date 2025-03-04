---
title: Use a managed identity for Azure Load Testing
titleSuffix: Azure Load Testing
description: Learn how to enable managed identity for Azure Load Testing and use it to read secrets from Azure Key Vault.
services: load-testing
ms.service: load-testing
ms.author: ninallam
author: ninallam
ms.date: 11/30/2021
ms.topic: how-to
---

# Use managed identities for Azure Load Testing Preview

This topic shows how you can create a managed identity for an Azure Load Testing Preview resource and how to use it to read secrets from Azure Key Vault.

A managed identity from Azure Active Directory (Azure AD) allows your resource to easily access other Azure AD-protected resources such as Azure Key Vault. The identity is managed by the Azure platform. For more about managed identities in Azure AD, see [Managed identities for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview).

Azure Load Testing supports only system-assigned identity. The system-assigned identity is associated with your Azure Load Testing resource and is removed when your resource is deleted. A resource can only have one system-assigned identity.

> [!IMPORTANT]
> Azure Load Testing is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Prerequisites  

- An Azure account with an active subscription. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.  

- An Azure Load Testing resource. If you need to create an Azure Load Testing resource, see the quickstart [Create and run a load test](./quickstart-create-and-run-load-test.md).

## Add a system-assigned identity  

To add a system-assigned identity for your Azure Load Testing resource, you need to enable a property on the resource. You can set this property by using the Azure portal, or by using an Azure Resource Manager (ARM) template.

### Set a system-assigned identity by using the Azure portal

To set up a managed identity in the portal you'll first create an Azure Load Testing resource, and then enable the feature.

1. Navigate to your Azure Load Testing resource in the [Azure portal](https://portal.azure.com).

1. Select **Identity**.

1. Switch on the system assigned identity, and then select **Save**.

    :::image type="content" source="media/how-to-use-a-managed-identity/system-assigned-managed-identity.png" alt-text="Screenshot that shows how to turn on system assigned managed identity for Azure Load Testing.":::

### Set a system-assigned identity by using an ARM template

An Azure Resource Manager template can be used to automate deployment of your Azure resources. Any resource of type `Microsoft.LoadTestService/loadtests` can be created with an identity by including the following property in the resource definition:

```json
"identity": {
    "type": "SystemAssigned"
}
```

Adding the system-assigned type tells Azure to create and manage the identity for your resource. For example, an Azure Load Testing resource might look like the following:

```json
{
    "type": "Microsoft.LoadTestService/loadtests",
    "apiVersion": "2021-09-01-preview",
    "name": "[parameters('name')]",
    "location": "[parameters('location')]",
    "tags": "[parameters('tags')]",
    "identity": {
        "type": "SystemAssigned"
    }
}
```

When the resource is created, it gets the following additional properties:

```json
"identity": {
    "type": "SystemAssigned",
    "tenantId": "<TENANTID>",
    "principalId": "<PRINCIPALID>"
}
```

The `tenantId` property identifies which Azure AD tenant the identity belongs to. The `principalId` is a unique identifier for the resource's new identity. Within Azure AD, the service principal has the same name as the Azure Load Testing resource.

## Grant access to Azure Key Vault

Managed Identity allows the Azure Load testing resource to access other Azure resources. In this section, you'll grant the Azure Load Testing service access to read secret values from an Azure Key Vault.

If you don't have an Azure Key Vault yet, see [Azure Key Vault quick-start](/azure/key-vault/secrets/quick-create-cli) to create it.

1. Open your Azure Key Vault resource in the Azure portal.

1. Under **Settings**, select **Access Policies**, and then **Add Access Policy**.

1. Select **Get** in the **Secret permissions**.

    :::image type="content" source="media/how-to-use-a-managed-identity/key-vault-add-policy.png" alt-text="Screenshot that shows how to add an access policy to Azure Key Vault.":::

1. Select **Select principal** and select your Azure Load Testing resource system-assigned principal.

    The name of the system-assigned principal is the same name as the Azure Load Testing resource.

1. Select **Add**.

You've now granted access to your Azure Load Testing resource to read the secret values from the Azure Key Vault.

## Next steps

- Parameterize a load test using secrets, see [Parameterize a load test](./how-to-parameterize-load-tests.md).

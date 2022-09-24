# Access Storage Blob with managed identity

This sample shows how to access Storage Blob with managed identity in `Azure Spring Cloud`.

You need include [ManagedIdentityCredentialBuilder](https://docs.microsoft.com/en-us/java/api/com.azure.identity.managedidentitycredentialbuilder?view=azure-java-stable) and [BlobServiceClientBuilder](https://docs.microsoft.com/en-us/java/api/com.azure.storage.blob.blobserviceclientbuilder?view=azure-java-stable) in your code. In this sample project, you could refer to [MainController.java](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/blob/master/managed-identity-storage-blob/src/main/java/com/microsoft/azure/MainController.java#L37). 

## Prerequisite

* [JDK 8](https://docs.microsoft.com/en-us/azure/java/jdk/java-jdk-install)
* [Maven 3.0 and above](http://maven.apache.org/install.html)
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) or [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview)
* An existing Storage account. If you need to create a Storage account , you can use the [Azure Portal](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal) or [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create)

## How to run 

1. Run `mvn clean package` after specifying the storage account and container in [application.properties](./src/main/resources/application.properties).
3. Create an instance of Azure Spring Cloud.
    ```
    az spring-cloud create -n <resource name> -g <resource group name>
   
    az spring create -n azsprcld -g springtest
    ```
4. Create an app with public domain assigned.
    ```
    az spring-cloud app create -n <app name> -s <resource name> -g <resource group name> --is-public true 
   
    az spring app create -n azsprgapp -s azsprcld -g springtest --is-public true 
    ```
5. Enable system-assigned managed identity for your app and take note of the principal id from the command output.
   ```
   az spring-cloud app identity assign -n <app name> -s <resource name> -g <resource group name>
   
   az spring app identity assign -n manid-azsprgapp -s azsprcld -g springtest 
   
   martinmalm@Martins-MacBook-Pro managed-identity-storage-blob % az spring app identity assign -n azsprgapp -s azsprcld -g springtest
   Assign managed identities without "system-assigned" or "user-assigned" parameters is obsolete, will only enable system-assigned managed identity, and will not be supported in a future release.
   Start to enable system-assigned managed identity.
   {
   "id": "/subscriptions/c6998809-fb9b-4cc3-944e-aa8a00d4e88c/resourceGroups/springtest/providers/Microsoft.AppPlatform/Spring/azsprcld/apps/azsprgapp",
   "identity": {
   "principalId": "c7c944a4-b0d7-4377-92f2-3005009a0efa",
   "tenantId": "44991835-87d7-4239-9df6-fc667fce766e",
   "type": "SystemAssigned",
   "userAssignedIdentities": null
   },

   ```
6. Grant permission of Storage Account to the system-assigned managed identity.
    ```
    az role assignment create --assignee <principal-id-you-got-in-step5> --role "Storage Blob Data Contributor" --scope <resource-id-of-storage-account>
   
    az role assignment create --assignee c7c944a4-b0d7-4377-92f2-3005009a0efa --role "Storage Blob Data Contributor" --scope "/subscriptions/c6998809-fb9b-4cc3-944e-aa8a00d4e88c/resourceGroups/testapper_group/providers/Microsoft.Storage/storageAccounts/mamastorageaccount"
    ```
7. Deploy app with jar.
    ```
    az spring-cloud app deploy -n <app name> -s <resource name> -g <resource group name> --jar-path ./target/asc-managed-identity-storage-blob-sample-0.1.0.jar
   
    az spring-cloud app deploy -n azsprgapp -s azsprcld -g springtest --jar-path ./target/asc-managed-identity-storage-blob-sample-0.1.0.jar
    ```
8.  Verify app is running. Instances should have status `RUNNING` and discoveryStatus `UP`.
    ```
    az spring-cloud app show -n <app name> -s <resource name> -g <resource group name>
    
    az spring-cloud app show -n azsprgapp -s azsprcld -g springtest

9. Verify sample is working. The url is fetched from previous step.
    ```
    # Upload data to blob
    curl -X PUT {url}/blob/{blob-name}?content={value}
   
   curl -X POST http://localhost:8080/blob/ablob?content=value

    # Get the content of blob-name 
    curl {url}/blob/{blob-name}
    # return the blob content you just uploaded before
    ```
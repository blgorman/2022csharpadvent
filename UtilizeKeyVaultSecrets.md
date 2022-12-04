# Utilize Secrets in your C# projects

This blog post is created for the 2022 [C# Advent Calendar Christmas 2022 event](https://csadvent.christmas/).  I'd like to thank []() for the opportunity to contribute.

The focus of this post will be a demonstration on how to utilize C# as a developer to keep your secrets safe, whether it's locally on your machine or if you are reading the value from a secret store such as Azure Key Vault.

If you want to work along, you'll need an Azure subscription for deployment of the App Service and storing and retrieving the key at Azure Key Vault.  The code for the final solution will be included in this repository.  Alternatively, creation of the code should be pretty trivial and you would likely be able to just create this as you read along.

- [Getting a trial subscription at Azure](https://azure.microsoft.com/free/search/)

## Getting Started

To get started, let's first just build a simple project that will be able to leverage a secret. Before building in the Azure Key Vault, we can just make the project read from the appsettings.json file and have that get a value that will ultimately be replaced with a secret from Key

### Create a simple MVC Web Project

For this demonstration, I'm using a default MVC application.  Code in other projects would be similar other than establishing injection and other trivial changes that happen in non-MVC projects.

1. Create the project.

    To create the project, run the following command:

    ```c#
    dotnet new mvc
    ```  

    !["running dotnet new mvc"](/images/image0001-dotnet-new-mvc.png)  

1. Add a secret value to the `appsettings.json` file.

    Add the secret to the `appsettings.json` file

    ```json
    "ImportantSecrets": {
        "API_KEY": "27dbd518-9f2b-493a-b814-f3b066facea1"
    }
    ``` 

1. Get the configuration injected into the Home Controller

    Add the injection of the configuration into the Home Controller

    ```cs
    private readonly IConfiguration _configuration;

    public HomeController(ILogger<HomeController> logger, IConfiguration configuration)
    {
        _logger = logger;
        _configuration = configuration;
    }
    ```  

1. Retrieve the secret from `appsettings.json`.

    Add the following code to the `HomeController.cs` file:

    ```cs
    public IActionResult Index()
    {
        var apiKey = _configuration["ImportantSecrets:API_KEY"];
        ViewData["API_KEY"] = apiKey;
        return View();
    }
    ```  

1. Display the secret on the `Index.cshtml` file.

    On the Index.cshtml file for the Home controller, add the following html and Razor code:

    ```html
    <div>
        <h1>API_KEY</h1>
        <span>@ViewData["API_KEY"]</span>
    </div>
    ```  

1. Run the application

    This is a great time to run the program to make sure that the secrets and configuration settings are working as expected.

    !["The secret is retreived and displayed on the home page"](/images/image0002-the-secrets-are-working.png)

## Switch to local user secrets

In order to protect your information locally, you should utilize the built-in local user secrets for .Net applications.  Moving the secret to the user secrets will keep you from accidentally committing a secret value into your GitHub repository.

1. Add user secrets to the project.

    To add user secrets to the project you can run some .NET commands, or you can just right-click and add them to your project. 

    !["Adding user secrets in the project"](/images/image0003-adding-user-secrets.png)  

1. Add the secret to the `usersecrets.json` project.

    To make this clear that the value is coming from secrets, add the following code into the `usersecrets.json` file:

    ```json
    {
        "ImportantSecrets:API_KEY":     "local-secrets-199f893a-336d-427b-a427-64b039ea293c"
    }
    ```  

    >**Note:** The nested nature of the key is preserved using the `:` character in the key.


1. Run the project.  What happens?

    The value from the `usersecrets.json` file has super-ceded the value in the local `appsettings.json` file.  

    !["The secrets file value supercedes the local appsettings.json"](/images/image0004-usersecrets-leveraged-as-expected.png)  

    If, for some reason, your secret is not shown, then consider removing or commenting the value from the `appsettings.json` file.

## Deploy the solution to Azure

To get the solution deployed into Azure, consider using the included YAML file to deploy.  In that file, you'll need to change the application name to map to your application and you'll need to make sure to add the publish settings into your github actions secrets file.

1. Create a new free app service at Azure

    The `F1` tier of the Azure App Service is free, and you can leverage it for this deployment.

    Log into your Azure portal and deploy a new `F1 - Free` app service plan on an azure app service.

    [Azure cloud shell](https://shell.azure.com)  

    Create a resource group:

    ```bash
    rgName=csadvent2022-workingwithsecrets
    loc=centralus
    az group create -n $rgName -l $loc
    ```   

    !["Creating the resource group"](/images/image0005-creatingrg.png)  

    Create the app service plan:

    ```bash
    planName=csadvent2022simpleweb
    sku=F1
    az appservice plan create -n $planName -g $rgName --sku $sku
    ```  

    !["Creating the app service plan"](/images/image0006-create-app-service-plan.png)

    After the plan is created, create the app service

    ```bash
    appServiceName=csadvent2022secrets-web
    az webapp create -n $appServiceName -g $rgName --plan $planName --runtime "dotnet:7"
    ```  

    !["Creating the app service"](/images/image0007-creatingtheappservice.png)  

1. Set up CI/CD on the app 

    If you are just using code from your machine, you could right-click and publish.

    If you want to deploy from a public GIT repo, just use the deployment center and point to the public GitHub URL.

    If you use the deployment center, you can also just connect to your own GitHub account and let Azure build the CI/CD.

    In the deployment center, select your GitHub repository and allow Azure to connect and deploy with the default CI/CD action for the project.

    !["Using the deployment center to set up CI/CD"](/images/image0008-usingthedeploymentcenter.png)  

1. Update CI/CD for Ubuntu

    Ubuntu builds at GitHub will run much more quickly than a windows build agent.  Update the GitHub action YAML to use the ubuntu-agent:

    !["Changing to the ubuntu agent"](/images/image0009-switch-to-ubuntu.png)  

1. Review the deployment

    Open the site after it deploys.  The original secret from the `appsettings.json` file is displayed.

    !["The app is deployed but the original secret is displayed"](/images/image0010-the-deployed-app-shows-appsettings-secret.png)

1. Update the configuration

    Override the configuration on the deployed application to show the secret from the configuration for the app service:

    !["Update App Service Configuration"](/images/image0011-appsettingsconfiguration-updated.png)  

    With the configuration updated, the new value shows as expected:

    !["The new value shows as expected"](/images/image0012-theupdatedsettingshows.png)  

## Place the secret in an Azure Key Vault

To keep the secret safe, the secret should be placed into an Azure Key Vault.  Once the key is in the Vault, the application can read the value as expected.

1. Create the key vault

    To store the secrets at Azure Key Vault, you'll need to create a vault.  To create the vault, run the following commands:

    ```bash
    kvName=csadvent2022vault
    az keyvault create --location $loc --name MyKeyVault --resource-group $rgName 
    ```  

    !["creating a keyvault"](/images/image0013-creating-a-keyvault.png)  

1. Create the Key Vault Secret

    You can create the KeyVault secret via the portal or via the CLI.  For this demonstration, we're already in the CLI so let's add it that way then just review it in the portal.

    ```bash  
    az keyvault secret set --vault-name $kvName --name "API-KEY" --value "vault-secret-59adf879-2722-410a-b8f9-97c42e03fc73"
    ```  

    Then view the secret

    ```bash
    az keyvault secret show --name "API-key" --vault-name $kvName --query "value"
    ```  

    !["Set and Get a vault secret with the az cli commands"](/images/image0014-setandgetvaultsecret.png)

    >**Note:** The Secret URI is shown as the ID in the image/output.  This URI also includes the secret version as part of the URI.

    You can see secrets and values in the Azure portal:

    !["Reviewing the secret in the vault"](/images/image0015-reviewsecretinthevault1.png)  

    Drill in to see more info:

    !["Drill in to get the secret URI and see more information"](/images/image0016-reviewsecretsinvault2.png)  

    Get the Secret URI For use from the App Service.

1. Use the secret from the App Service

    In order to use the secret from the App Service, you need to wrap the value with:

    ```text
    @Microsoft.KeyVault(SecretUri=.....)
    ```  

    Replace the `.....` with the URI copied above, such as:

    ```text
    @Microsoft.KeyVault(SecretUri=https://csadvent2022vault.vault.azure.net/secrets/API-KEY/1e3698837c4e460b8b6eab48ff2e834a)
    ```  

    Place the value in the Configuration value for the API key in the App Service.  Note that you didn't have to name the secret the same as the value for the key.

    !["Setting the configuration to utilize the Key vault secret](/images/image0017-configurationtokeyvault.png)

    >**Note:** The key vault reference currently will not work.  The reference has a red X and if you view it in the app you would see the text entry:

    !["The key is currently not retrievable"](/images/image0018-theapikeysecretisntcurrentlyretrievable.png)  

1. Set the identity for the App Service

    For the app service to work, you'll need to give it a system-managed identity, and then you'll need to give it permission in the vault.

    To enable the system-managed identity, on the App Service in the Portal, under the `Identity` left-navigation menu, select `On` and then hit `Yes` to save the identity.  

    !["Giving the App service a system-managed identity"](/images/image0019-creatingsystemmangedidentity.png)  

    The identity will allow the App Service to be used in RBAC on the KeyVault policies.

    Once the identity is saved, copy the object id from the identity blade

    !["Get the object id for use in RBAC"](/images/image0020-gettingtheobjectid.png)  

1. Give the app service permission on the Key Vault

    Once the App Service has a system-managed identity, the identity can be used in RBAC to give `GET` permission on the secrets from the vault.

    Navigate to the Key Vault and then choose the `Access Configuration` left-navigation menu.  On the menu, select `Go to Access Policies` to open the policies blade.

    Select `+ Create` to begin creating a policy.

    On the `Basics` tab select only the `Secrets -> Get` option.  Everything else should be unchecked.

    !["Select only the GET permission"](/images/image0021-addthesecretgetpermissiononly.png)  

    On the next blade, enter the object id retrieved above, and then select the identity.  

    !["Select the principal by object ID"](/images/image0022-usetheobjectidforpermissions.png)  

    Complete the assignment.  When done, it should be listed on the main policies blade.

    !["Policies now show the get permission for the app service"](/images/image0023-getSecretPermissionAdded.png)  


1. Review the application configuration

    After setting the configuration above and also giving permissions with the ID as above, wait a couple of minutes for the permissions to propagate, and go back to the App Service. 

    Review the configuration to see if a green checkmark has been added to the configuration setting.

    !["Key Vault Secret is now showing with a green checkmark"](/images/image0024-KeyVaultIsAuthorizedAndSecretIsRetrieved.png)  

1. Review the Application

    Navigate to the application, and you will be able to see the Key Vault Secret displayed from the Web application.

    !["Vault Secret Shown on Web App"](/images/image0025-vault-secret-shown-on-web-app.png)  

## What about C#

I know what you're thinking:  This is a C# event, not really an Azure event, and this post has shown very little C#.  For that reason, to finish this up, let's do a couple of quick C# commands against the vault.

In order for this to work, your user account will need to have the appropriate permissions on the Key Vault.





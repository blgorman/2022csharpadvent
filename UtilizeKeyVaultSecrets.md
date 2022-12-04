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
    planName=csadvent2022web
    sku=F1
    az appservice plan create -n $planName -g $rgName --sku $sku
    ```  

    !["Creating the app service plan"](/images/image0006-create-app-service-plan.png)

    After the plan is created, create the app service

    ```bash
    appServiceName=csadvent2022secrets-web
    az webapp create -n $appServiceName -g $rgName --plan $planName
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





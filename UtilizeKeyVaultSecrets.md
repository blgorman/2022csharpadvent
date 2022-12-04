# Utilize Secrets in your C# projects

This blog post is created for the 2022 [C# Advent Calendar Christmas 2022 event](https://csadvent.christmas/).  I'd like to thank []() for the opportunity to contribute.

The focus of this post will be a demonstration on how to utilize C# as a developer to keep your secrets safe, whether it's locally on your machine or if you are reading the value from a secret store such as Azure Key Vault.

If you want to work along, you'll need an Azure subscription for deployment of the App Service and storing and retrieving the key at Azure Key Vault.  The code for the final solution will be included in this repository.  Alternatively, creation of the code should be pretty trivial and you would likely be able to just create this as you read along.

- [Getting a trial subscription at Azure](https://azure.microsoft.com/free/search/)

## Getting Started

To get started, let's first just build a simple project that will be able to leverage a secret. Before building in the Azure Key Vault, we can just make the project read from the appsettings.json file and have that get a value that will ultimately be replaced with a secret from Key

### Create a simple MVC Web Project

For this demonstration, I'm using a default MVC application.  Code in other projects would be similar other than establishing injection and other trivial changes that happen in non-MVC projects.

Run the following command:

```c#
dotnet new mvc
```  
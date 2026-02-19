---
title: Finishing the Cloud Resume Challenge
image: cloudResumeHeader.webp
date: 2024-10-04 12:00:00 
categories: [cloud, resume]
tags: [cloud,resume]
---

## Returning to the Challenge
My process in expanding my skill set to the cloud and specifically **Azure** has been a long and rewarding experience. Learning new skills and concepts, gaining hands-on experience through interactive training and labs, as well as obtaining my Azure certification (first of many).

This is the second post in the series of posts revolving around this challenge, I last left off in February earlier in the year. At that point the project was mapped to a custom domain name and it had a simple JavaScript site counter which worked off the local storage in the user's browser. 

# Recap of Steps Taken
1. ### Certification
    Acheived the AZ-900 (Azure Fundamentals) certification whcih is an introductory certification that orients someone with Azure. It offers a great lay of the land of the possibilities with the Cloud Platform. This is the first step I took. 
2. ### HTML
    Secondly I converted my resume into html. 
3. ### CSS
    Next, I created a CSS file for my website's style.

4. ### Static website: 
    I deployed my resume/portfolio site online as an <a href="https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website">Azure Storage Static Site,</a> this was pretty easy to do as the service just needs to be enabled with a index HTML file and an error document path.

    ![image of static site](staticsite.png)

5. ### HTTPS:
    I converted the URL for the storage site from HTTP to [HTTPS](https://www.cloudflare.com/learning/ssl/what-is-https/) for security. I needed to use Azure CDN for this and use **CDN Managed** as the Certificate management type and TLS version 1.2.

6. ### DNS:
    I pointed a custom domain name from 
    [godaddy.com](https://www.godaddy.com/) to my configured Azure CDN endpoint. So instead of navigating to a URL like `mysite.z13.web.core.windows.net/` it is much simpler to access from other devices or present the project itself.

7. ### Javascript:
    I included javascipt code to include a visitor counter that displays how many people have accessed the site. Here is a [helpful tutorial](https://www.codecademy.com/learn/introduction-to-javascript) that I used.

    ```javascript
    script src="https://ajax.googleapis.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
        <script>
        $(document).ready(function() {
            var count = localStorage.getItem('visitorCount');
            if (!count) {
            count = 0;
            } else {
            count = parseInt(count);
            }
            count++;
            $('#visitorCount').text(count);
            localStorage.setItem('visitorCount', count);
        });
        </script>
    ```

# Website Revamp and Structural Enhancements
After taking a break, I decided to give my website a much-needed facelift. Below is a preview of the refreshed look:

![image of static site](newsite.png)

As part of this update, I restructured the site by separating the backend (API & testing) from the frontend (CSS, images, HTML, JavaScript) to improve maintainability and scalability. Additionally, I upgraded the domain from kooz.store to a more professional looking domain name *ryankuczer.com*, which aligns better with my current objectives and projects.

## Starting the Website Counter
One of the challenges I faced early on was implementing a visitor counter. Back when I was in college, this proved to be tricky. However, with my current experience in the industry and increased familiarity with Azure, I was able to achieve this much more efficiently.

So to start off in my HTML I included an anchor tag for the counter `<a id="counter"></a>`. 

Next, I developed a simple JavaScript function that retrieves the visit count from the API (which I built later in the project). When the page loads, it triggers the `getVisitCount` function, which updates the counter based on the data provided by the API.

```javascript
window.addEventListener('DOMContentLoaded', (event) => {
    getVisitCount();
});

const functionAPIUrl = 'FUNCTIONAPIURL'

const getVisitCount = () => {
    fetch(functionAPIUrl)
        .then(response => response.json())
        .then(data => {
            console.log("Website called function API.");
            console.log("Count data received:", data);
            document.getElementById("counter").innerText = data.count; // Display the count
        })
        .catch(error => {
            console.error("Error fetching the count:", error);
        });
};
```

## Building the Backend

### Cosmos DB
Since I’m using Azure and prefer to keep everything under one cloud service provider, I set up a Cosmos DB account. For the API, I chose SQL and opted for a `serverless` capacity mode, which is the most cost-effective option.

I created a database, then a container, and finally an item in the database with an `id` of 1 and a `count` of 0. 

![alt text](image.png)

### Using Azure Functions
To connect Cosmos DB with the frontend, I developed an Azure Function. In the past, I built functions directly within the Azure portal. However, this time I created the function in Visual Studio Code, allowing me to test everything locally before deploying.

I chose `C#` as the programming language, with `.NET` as the runtime, using the in-process model and an `HttpTrigger` to trigger the function.

I then integrated the Cosmos DB trigger, utilizing both [input](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2-input?tabs=python-v2%2Cisolated-process%2Cnodejs-v4%2Cextensionv4&pivots=programming-language-csharp) and [output](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2-output?tabs=python-v2%2Cisolated-process%2Cnodejs-v4%2Cextensionv4&pivots=programming-language-csharp) bindings. The Microsoft documentation linked were invaluable in completing this part of the project.

**Important Note:** When using Azure Functions locally, it’s essential to install the necessary VS Code extensions for Azure, as well as the external `NuGet` package for the .NET CLI, especially when using `C#`.

![Input Binding](cosmosdbinput.png)

Here is the code used to retrieve the count from the database and update it by 1. The `AzureResumeConnectionString` was set in my `local.settings.json` for testing at the time.

```C#
    public static class UpdateCountFunction
    {
        [FunctionName("UpdateCount")]
        public static async Task<HttpResponseMessage> Run(
            [HttpTrigger(AuthorizationLevel.Function, new[] { "get", "post" }, Route = null)] HttpRequest req,
            [CosmosDB(
                databaseName: "AzureResume",
                containerName: "Counter",
                Id = "1",
                PartitionKey = "1",
                Connection = "AzureResumeConnectionString")] JObject document,
            [CosmosDB(
                databaseName: "AzureResume",
                containerName: "Counter",
                Connection = "AzureResumeConnectionString")] IAsyncCollector<JObject> updatedDocumentCollector,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function to update count in Cosmos DB.");

            // Increment the count
            int currentCount = (int)document["count"] + 1;
            document["count"] = currentCount;

            // Add the updated document to the collector to replace the item in Cosmos DB
            await updatedDocumentCollector.AddAsync(document);

            // Log the updated count
            log.LogInformation($"Count updated to {currentCount}");

            // Create and return a properly formatted JSON response
            var response = new HttpResponseMessage(HttpStatusCode.OK)
            {
                Content = new StringContent($"{{\"count\": {currentCount}}}", System.Text.Encoding.UTF8, "application/json")
            };
            response.Headers.Add("Access-Control-Allow-Origin", "*"); // Allow all origins
            response.Headers.Add("Access-Control-Allow-Methods", "GET, POST");
            response.Headers.Add("Access-Control-Allow-Headers", "Content-Type");

            return response;
        }
    }
```
At this point the count variable updates by 1 locally in my html successfully, next is to deploy to Azure.

### Deploying to Azure
Since I had my CDN and custom domain already prepared for this section all that was needed was to deploy the function to Azure and update the javascript code I mentioned earlier in this article.

Once the function app was deployed I added a variable in the function for my `AzureResumeConnectionString` to connect to my Cosmos DB, later on I will use Azure Key Vault for this instead. 

![Azure Function](httptrig.png)

Now, the website is up and running with my custom domain and the count updates correctly.

### Infrastructure as Code
Instead of configuring Azure resources manually through the portal, I defined them using an Azure Resource Manager (ARM) template — a practice known as Infrastructure as Code (IaC).

I exported my existing Resource Group from the Azure Portal, which generated a `template.json` and `parameters.json` representing my Azure Function and CosmosDB configuration. These files are stored in the `infra/` folder of my repository.

To automate deployment, I created a GitHub Actions workflow (`infra.yml`) that triggers whenever changes are pushed to the `infra/` folder. It uses the `azure/arm-deploy` action to deploy the template directly to Azure:

```yaml
name: deploy_infrastructure
on:
  push:
    branches: [ main ]
    paths:
      - 'infra/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    - name: Deploy ARM Template
      uses: azure/arm-deploy@v1
      with:
        resourceGroupName: ResumeSite
        template: ./infra/template.json
        parameters: ./infra/parameters.json
```

This ensures my infrastructure is version-controlled, repeatable, and not dependent on manual portal configuration.

#### Use Cases for ARM
- **Repeatable Deployments** — Spin up identical dev, staging, and production environments from a single template
- **Disaster Recovery** — Redeploy all infrastructure in minutes if something goes wrong
- **Team Collaboration** — Infrastructure lives in source control so changes are tracked like code
- **Compliance & Auditing** — Every infrastructure change is logged, who made it and when
- **Cost Control** — Tear down environments when not in use, redeploy later to avoid idle costs

### Continuous Integration and Development
![alt text](image1.png)

*Why would I want to purge the CDN endpoint everytime a change is made to my website?*

This is exactly what CI/CD pipelines are used for. So I moved all of my code to GitHub and created a `github/workflows` directory as well as a frontend and backend `.yml` file. After, I added my connection strings as secrets to the GitHub repository so they do not have to be added in clear text. 

To do this I had to add a role to my specific resource group in Azure with the storage account via the command `az ad create-for-rbac....`. And then that `JSON` response is added to my GitHub Repository as a secret.

#### Frontend Workflow
I wanted to separate the workflows so both will not run everytime a change is made in the repository. So the frontend workflow only runs when there is a change in my `Frontend` folder, here it is below...
```yml
name: deploy_frontend
#Deploys when a push is made from the front end folder.
on:
    push:
        branches: [ main ]
        paths:
        - 'frontend/**'
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: azure/login@v1
      with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Upload to blob storage
      uses: azure/CLI@v1
      with:
        inlineScript: |
            az storage blob upload-batch --account-name STORAGEACCNAME --auth-mode login -d '$web' -s frontend/ --overwrite
    - name: Purge CDN endpoint
      uses: azure/CLI@v1
      with:
        inlineScript: |
           az cdn endpoint purge --content-paths  "/*" --profile-name "MYPROFILENAME" --name "MYNAME" --resource-group "GROUPNAME"

  # Azure logout
    - name: logout
      run: |
            az logout
      if: always()
```
This workflow:
1. Looks for any change in the `frontend` folder
2. Login to Azure with the provided credential
3. Upload new content to blob storage
4. Purge CDN endpoint to reflect new changes on the website
5. Log out

#### Unit Tests
Before adding the backend workflow I needed to add unit tests to my code, otherwise nothing would be executed in the workflow.

**Why Unit Testing?**
* Saves time debugging
* Allows for mre efficient code
* Improves Documentation
* Reduces errors during deployement

So, I used `dotnet` testing. Some commands used were `dotnet new xunit` and `dotnet add reference ../api/api.csproj`. Here is my unit test for the counter function: 
```cs
namespace tests
{
    public class TestCounter
    {
        private readonly ILogger logger = TestFactory.CreateLogger();

        [Fact]
        public async Task UpdateCount_Should_Increment_Counter()
        {
            // Arrange
            var initialCount = 5;
            var document = new JObject
            {
                ["id"] = "1",
                ["count"] = initialCount
            };

            var collector = new TestAsyncCollector<JObject>();
            var request = TestFactory.CreateHttpRequest();

            // Act
            var response = await UpdateCountFunction.Run(
                request,
                document,
                collector,
                logger);

            // Assert
            Assert.Equal(HttpStatusCode.OK, response.StatusCode);
            Assert.Single(collector.Items);
            Assert.Equal(initialCount + 1, (int)collector.Items[0]["count"]);
        }
```
We can see it is straightforward, the `initialCount` is set to 5 and the test *asserts* that the new count must be `initialCount` + 1. 

I also had tests for the function returning proper JSON content, and the correct headers. It is important to note I also had helper files for the testing they are: 
* ListLogger.cs
* LoggerType.cs
* Nullscope.cs
* TestFactory.cs

![alt text](image2.png)

After all of the tests passed I was able to start the backend workflow.

#### Backend Workflow
```yml
name: deploy_backend

on:
  push:
      branches: [main]
      paths:
      - 'backend/**'

env:
  AZURE_FUNCTIONAPP_NAME: 'FUNCTIONNAME'   # set this to your function app name on Azure
  AZURE_FUNCTIONAPP_PACKAGE_PATH: 'backend'       # set this to the path to your function app project, defaults to the repository root
  DOTNET_VERSION: '6.0'                   # set this to the dotnet version to use (e.g. '2.1.x', '3.1.x', '5.0.x')

jobs:
  build-and-deploy:
    runs-on: windows-latest
    environment: dev
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@v3

    - name: 'Login with Azure Cli'
      uses: azure/login@v1
      with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Setup DotNet ${{ env.DOTNET_VERSION }} Environment
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: 'Resolve Project Dependencies Using Dotnet'
      shell: pwsh
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/api'
        dotnet build --configuration Release --output ./output
        popd

    - name: 'Run Unit Test'
      shell: pwsh
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/tests'
        dotnet test

    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/api/output'
```

We can see here this workflow performs the following:
1. Looks for any change in the `backend` folder
2. Set environment variables for the Function App name, package path, and dotnet version
3. Login to Azure with the provided credential
4. Setup dotnet environment
5. Resolve dependencies
6. Perform the written unit tests
7. Run the Azure function action

## Conclusion
At this point, I have a much more robust and complicated (on purpose) resume website, which has allowed me to learn various different tools in Azure. This project has enhanced my technical skills and deepened my understanding of cloud architecture and deployment strategies.

Working with services like Azure Functions, and Cosmos DB has been invaluable. I’ve also gained hands-on experience with CI/CD pipelines, using GitHub actions to streamline my development process and improve collaboration.

Reflecting on this challenge, I recognize how it has pushed me to tackle complex problems and seek innovative solutions. The knowledge I’ve gained will serve as a strong foundation for my future endeavors in cloud technologies. I’m excited to continue exploring and expanding my skills in this ever-evolving field.


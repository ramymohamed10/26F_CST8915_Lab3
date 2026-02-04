# Lab 3 - CST8915 Full-stack Cloud-native Development: Deploying the Algonquin Pet Store on Azure

Welcome to Lab 3 of the **CST8915 Full-stack Cloud-native Development** course. In this lab, you will set up a RabbitMQ server on an Azure Virtual Machine (VM) and deploy the `order-service`, `product-service`, and `store-front` (from the repos created in Lab 2) using Azure Web App Service and Azure Static Web Apps.

In Lab 2, you refactored the Algonquin Pet Store and applied the first four factors of the 12-Factor App, and deployed each service on its own Azure VM. In this lab, you’ll take the next step: moving away from VM-based deployments and into Azure Platform-as-a-Service (PaaS) offerings. Instead of running everything on separate VMs, you’ll use Azure Web App Service for the backend APIs and Azure Static Web Apps for the frontend. This shift demonstrates how cloud-native applications benefit from managed services: less infrastructure to manage, more scalability, and easier integration.

## Lab Objectives:
- Create and configure an Azure VM as a RabbitMQ server.
- Rewrite the `product-service` in a programming language natively supported by Azure App Service.
- Deploy the `order-service` and `product-service` on Azure Web App Service.
- Deploy the `store-front` on Azure Static Web Apps.
- Integrate these services and ensure they work together as expected.

## Understanding Azure Web App and Azure Static Web Apps

### Azure Web App
- [Azure Web App](https://learn.microsoft.com/en-us/azure/app-service/overview) is a fully managed platform for building, deploying, and scaling web apps. It supports various programming languages and frameworks, including Node.js, Python, .NET, Java, and PHP. With Azure Web App, you can quickly deploy web applications without worrying about managing infrastructure, and you benefit from built-in security features, continuous integration/continuous deployment (CI/CD), and auto-scaling.

- For this lab, you'll use Azure Web App to deploy both the `order-service` and the rewritten `product-service`. These services are backend APIs that will handle business logic and communicate with the RabbitMQ server.

- To get started with Azure Web App, you can follow this [Quickstart guide](https://learn.microsoft.com/en-us/azure/app-service/quickstart-nodejs?tabs=windows) that provides step-by-step instructions on deploying a Node.js app (similar steps apply to other languages).

### Azure Static Web Apps
- [Azure Static Web Apps](https://learn.microsoft.com/en-us/azure/static-web-apps/overview) is a service that automatically builds and deploys full-stack web apps to Azure from a GitHub repository. It is optimized for static front-end frameworks like Vue.js, React, and Angular, and integrates seamlessly with backend APIs hosted on Azure Functions or other services.

- In this lab, you'll use Azure Static Web Apps to deploy the `store-front`, which is a Vue.js application. This service will also manage the CI/CD pipeline for your front-end code via GitHub Actions.

- To get started with Azure Static Web Apps, follow this [Quickstart guide](https://learn.microsoft.com/en-us/azure/static-web-apps/getting-started?tabs=vanilla-javascript) that provides detailed steps on deploying your first static web app.



## Lab Steps:

### Step 1: Create and Configure an Azure VM for RabbitMQ

For this step, refer to the instructions provided in previous labs where you set up RabbitMQ on an Azure VM. Ensure that your RabbitMQ instance is running and accessible, and that you have noted the RabbitMQ connection string, which will be used in the subsequent steps.

### Step 2: Rewrite the Product-Service

#### 2.1. Choose a Programming Language
- The original product-service was written in Rust. However, Rust is not natively supported by Azure App Service. You are required to rewrite the product-service in a programming language that is natively supported by Azure App Service.
- **Recommendation**: If you are undecided, we recommend rewriting the service in **Python**. Python is well-supported on Azure and is relatively easy to set up and deploy.

#### 2.2. Ensure Compliance with the 12-Factor App Methodology
- As you rewrite the product-service, make sure it complies with the first four factors of the [12-Factor App](https://12factor.net/) methodology, just as you did in Lab 2.

#### 2.3. Rewrite the Service
- Recreate the functionality of the original Rust-based product-service in the chosen programming language. Ensure the rewritten service maintains the same API endpoints and logic.


#### 2.4. Test Locally
- Before deploying, test the rewritten product-service locally to ensure it functions as expected and complies with the 12-Factor App methodology.

### Step 3: Deploy Order-Service and Product-Service on Azure Web App Service
#### 3.1. Create Azure Web Apps
- In the Azure Portal, navigate to "App Services" and create a new Web App for each service:
  - **Name**: `order-service-app` and `product-service-app`
  - **Runtime stack**: Choose Node.js for order-service and Python (or the appropriate stack) for the rewritten product-service.
#### 3.2. Edit `package.json` for Order-Service
  - In `package.json` of `order-service`, make sure that `scripts` section include only the followings:
    ```
    "scripts": {
      "test": "echo \"No tests available\""
    }
    ```
  - `package.json` of the `order-service` should be as followings:
    ```
    {
      "name": "order-service-algonquin-pet-store",
      "version": "1.0.0",
      "main": "index.js",
      "scripts": {
        "test": "echo \"No tests available\""
      },
      "keywords": [],
      "author": "",
      "license": "ISC",
      "description": "",
      "dependencies": {
        "amqplib": "^0.10.4",
        "cors": "^2.8.5",
        "dotenv": "^16.4.5",
        "express": "^4.19.2"
      }
    }
    ```
#### 3.3. Deploy the Services
- Use the Azure CLI or Azure portal to deploy the code from your GitHub repositories to these Web Apps.
- Ensure that the environment variables (e.g., RabbitMQ connection string, service ports) are set up in the Azure Web App settings.

### Step 4: Deploy Store-Front on Azure Static Web Apps
#### 4.1. Create an Azure Static Web App
- In the Azure Portal, navigate to "Static Web Apps" and create a new static web app.
  - Source: Choose GitHub as the deployment source.
  - Repository: Select the repository containing your store-front code.
  - Build Presets: Choose "Vue.js" if prompted.

#### 4.2. Configure Environment Variables in GitHub Actions
When we deployed the `store-front` in Lab 2, it could easily talk to the `order-service` and `product-service` by reading the `.env` file on the VM. But once we move the app to Azure Static Web Apps, there is no local .env file anymore: the app is built and deployed automatically using GitHub Actions. That means we need to tell GitHub Actions what backend services (and URLs) our `store-front` should connect to. We do this by defining `environment variables` in the GitHub Actions workflow file, so that during the build process, the `Vue.js` app **“bakes in”** the correct service URLs (for `order-service` and `product-service`). Without this step, the store-front would not know how to reach the backend services deployed on Azure.

Unlike backend services (such as `Node.js` or `Python` apps) that can read environment variables at runtime, a `Vue.js` (or any front-end JavaScript framework) application is different. When you build the store-front, all of its code is compiled into static HTML, CSS, and JavaScript files. Once deployed, these files are just served to the browser: they can’t dynamically read environment variables from the server at runtime because they don’t run on a server anymore, they run in the user’s browser.

This means the correct API URLs (`VUE_APP_ORDER_SERVICE_URL`, `VUE_APP_PRODUCT_SERVICE_URL`) must be **“baked into”** the code during the build stage. By adding environment variables to the GitHub Actions workflow, we ensure that when the static files are generated, they **already** contain the right configuration values pointing to your deployed services.


When deploying your `store-front` using Azure Static Web Apps and GitHub, a GitHub Actions workflow is automatically generated to manage the Continuous Integration/Continuous Deployment (CI/CD) pipeline. However, you need to ensure that the store-front application is correctly configured to communicate with the backend services (order-service and product-service) by including the necessary environment variables.

#### Step-by-Step Instructions:

1. **Locate the GitHub Actions Workflow File:**
    - After setting up your Azure Static Web App, navigate to your GitHub repository.
    - Locate the `.github/workflows` directory. Inside this directory, you will find the automatically generated GitHub Actions workflow file, typically named something like `azure-static-web-apps-<unique_id>.yml`.
2. **Modify the Workflow File:**
    - Open the workflow file and look for the `jobs:` section.
    - Under jobs:, find the `build_and_deploy_job:` and add an `env:` block to declare the environment variables.
3. **Add Environment Variables:**
    - Insert the following `env:` block into the `build_and_deploy_job:` section to set the necessary environment variables for your `Vue.js` application:

      ```
          env: 
            VUE_APP_ORDER_SERVICE_URL: https://<order-service-app>.azurewebsites.net/
            VUE_APP_PRODUCT_SERVICE_URL: https://<product-service-app>.azurewebsites.net/
      ```
     - Replace `<order-service-app>` and `<product-service-app>` with the actual names of your deployed services.

    - Now `build_and_deploy_job:` section should look like that:
      ```
      jobs:
        build_and_deploy_job:
          if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
          runs-on: ubuntu-latest
          name: Build and Deploy Job
          env: # Declare environment variables here
            VUE_APP_ORDER_SERVICE_URL: https://<order-service-app>.azurewebsites.net/
            VUE_APP_PRODUCT_SERVICE_URL: https://<product-service-app>.azurewebsites.net/
          steps:
            - uses: actions/checkout@v3
              with:
                submodules: true
                lfs: false
            - name: Build And Deploy
              id: builddeploy
              uses: Azure/static-web-apps-deploy@v1
              with:
                azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_<UNIQUE_IDENTIFIER> }}
                repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for GitHub integrations (i.e., PR comments)
                action: "upload"
                ###### Repository/Build Configurations - These values can be configured to match your app requirements. ######
                app_location: "/" # App source code path
                api_location: "" # Api source code path - optional
                output_location: "dist" # Built app content directory - optional
                ###### End of Repository/Build Configurations ######
      ```
4. **Save and Commit the Changes:**
    - After editing the workflow file, commit the changes to your repository. This will trigger the GitHub Actions workflow, deploying your updated store-front with the correct environment variables.

5. **Verify the Deployment:**
    - Once the deployment process completes, visit the URL provided by Azure Static Web Apps.
    - Ensure that your store-front is correctly interacting with the order-service and product-service via the URLs provided in the environment variables.

### Step 5: Testing and Validation
- After all services are deployed, access your store-front through the Static Web App URL.
- Place an order and ensure that it is processed correctly through RabbitMQ and the backend services.
- Monitor logs and the RabbitMQ management UI to ensure all components are communicating correctly.

## Submission

For this third lab, you will demonstrate your ability to deploy microservices on Azure PaaS and reflect on the differences between VM-based and managed-service deployments.

### What to Submit
1. **Demo Video (Max 5 minutes)**  
   - Record a short demo video showing your deployed application running on Azure Web App Service and Azure Static Web Apps, integrated with RabbitMQ on an Azure VM.  
   - At minimum, your demo should include:  
      - The store-front running in your browser, served from Azure Static Web Apps.
      - The deployed URLs of the order-service and product-service (Azure Web Apps) shown briefly in the Azure portal or terminal.
      - Confirmation that environment variables are configured:
        - Show environment variables set in the Azure Web App settings for the backend services.
        - Show environment variables declared in the GitHub Actions workflow for the store-front.
      - A demonstration that:
        - The store-front loads products from the product-service.
        - An order is placed through the order-service.
        - The order-service successfully communicates with RabbitMQ hosted on its dedicated VM (e.g., show message activity in the RabbitMQ Management UI).

2. **Reflection Questions**  
   Answer the following in your `README.md` file. Keep your answers **short but thoughtful** (≈1 short paragraph each).  
    1. What challenges did you encounter when configuring environment variables in the GitHub Actions workflow?
    2. How does deploying microservices on Azure Web App Service differ from running them locally?
    3. Why is it important to use environment variables for configurations in a cloud environment?

3. **GitHub Repository (Submission Repo)**  
   - You must create **one GitHub repository** for your lab submission.  
   - This submission repository must include:  
     - A `README.md` file with:  
       - The YouTube demo video link.  
       - Your written answers to the Reflection Questions.  
       - Links to the three service repositories you created in lab 2:  
         - `order-service` 
         - `product-service`  
         - `store-front`  
     - (Optional) Notes about setup challenges or lessons learned.  

### How to Submit
- Push your work to a **public GitHub repository** (the submission repository).  
- Submit the link to your submission repository as your final lab deliverable in **Brightspace**.  
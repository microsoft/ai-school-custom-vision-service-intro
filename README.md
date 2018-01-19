
# Getting started with Custom Vision Service

This lab will show you how to bring advanced ML vision capabilities to your applications with the Custom Vision Service. The service makes it easy to build custom image classifiers and provides APIs and tools to help you improve your classifier over time. The lab is split in two parts: first, we'll show you how to use the Custom Vision portal to build a classifier and then we'll show you how to do the same thing using the SDK. In both cases, we'll be building a classifier that can distinguish between a Surface Pro and a Surface Studio.

## Setup your environment (5 minutes)

### A) Set up your account

The Custom Vision portal **requires** either a work or school account (e.g. Office 365 accounts) or a personal Microsoft account (e.g. `@outlook.com`). You can create a new personal Microsoft account [here](https://signup.live.com) (signup.live.com) with an existing email address (or create a new one).

### B) Set up your machine

The first part of the lab, where the portal is used to build a classifier, only requires a modern web browser.

The second part of the lab, where the SDK is used to build a classifier, requires **Visual Studio 2017** on Windows. Get it [here](https://www.visualstudio.com/downloads/). While the SDK in preview is Windows-only, the underlying RESTful APIs can be accessed from any platform. Learn more about the training API [here](https://go.microsoft.com/fwlink/?linkid=865445) and the prediction API [here](https://go.microsoft.com/fwlink/?linkid=865446).

## Build a classifier (10 min)

### A) Create Custom Vision project

Before we can start training our classifier, we need to create a project in the Custom Vision portal. The Custom Vision Service is currently in preview so we don't need to provision a subscription in Azure (as is the case with most of the other Cognitive Services).

1. Open a browser and navigate to the [Custom Vision](https://customvision.ai/) (customvision.ai) portal.
1. **Log in** using a valid account (learn more above).
1. If this is your first time visiting the portal, it will request some permissions. Click the **Yes** button to agree (you can revoke these permissions later if necessary).
1. If this is your first time visiting the portal, it will also prompt you to agree with the terms and conditions. **Check** the box to indicate consent and then click the **I agree** button.
1. Click **New Project**.
1. Provide the required values:
    * Name: `customvisionlab`
    * Description: `Custom Vision Lab`
    * Domains: `General` (i.e. the default)
1. Click **Create Project**.

### B) Add Images

Training an image classifier requires example images. While our data set is relatively small and can be uploaded via a web browser, it's likely that production scenarios will require hundreds or thousands of images and thus are best uploaded via the API (covered later in the lab).

1. Click the **Training Images** tab at the top of the page.
1. Click the **Add images** button.
1. Click the **Browse local files** button.
1. Browse to `$/Training/Images/surface-pro` and select **all** of the images.
1. Click **Open**.
1. In the **Add some tags to this batch of images** field, enter the tag `surface-pro`.
1. Click on the **+** button next to the field.
1. Click the **Upload 15 files** button.
1. Wait for the upload to complete. This can take a few moments.
1. Click the **Done** button.
1. Click the **Add images** button.
1. Click the **Browse local files** button.
1. Browse to `$/Training/Images/surface-studio` and select **all** of the images.
1. Click **Open**.
1. In the **Add some tags to this batch of images** field, enter the tag `surface-studio`.
1. Click on the **+** button next to the field.
1. Click the **Upload 15 files** button.
1. Wait for the upload to complete. This can take a few moments.
1. Click **Done**.

### C) Train the classifier

Now that our data set is available, and each image correctly tagged, the training process can begin. This is a quick operation given the small size of our data set. The output is a classifier that can be used to make predictions. As more training data is added, we can create a new iteration of the classifier and analyze its performance.

1. Click on the green **Train** button in the top nav bar.
1. Wait for the training to complete.
1. Review the new classifier's precision and recall. Mouse over the tooltips to learn more about these concepts.

> It's recommended that at least 30 images of each class or category are included for a prototype classifier. In the interests of time, we're only providing 15 images for each class so we should expect slightly lower accuracy compared to a fully trained version.

### D) Test the classifier

The portal provides a test page for us to do a basic smoke test of our model with some new, unseen data. In a production scenario, the RESTful API is a more appropriate mechanism for doing bulk predictions (covered later in the lab).

1. Click on the **Quick Test** button in the top nav bar.
1. Click the **Browse local files** button.
1. Browse to `$/Prediction/Images/surface-pro` and select the `pro_test.jpg` image.
1. Click **Open**.
1. Wait for the classification to run and ensure it is correctly recognized as a `surface-pro`.
1. Click the **Browse local files** button.
1. Browse to `$/Prediction/Images/surface-studio` and select the `studio_test.jpg` image.
1. Click **Open**.
1. Wait for the classification to run and ensure it is correctly recognized as a `surface-studio`.
1. Close the **Quick Test** side bar by clicking on the **X** icon in the top right.
1. Click on the **Predictions** tag in the top nav bar.
1. Here we can see the new test data. If we click on an image, we can manually tag it and add it to our data set. This image will then be included next time we re-train our classifier, potentially leading to improved accuracy.

## Use the training API (5 min)

### A) Prepare the solution

Now that we've seen the Custom Vision Service in action, next we'll examine how we can use the API to complete the same process. Using the API helps us train and predict more efficiently. In order to accelerate the lab, we've prepared a basic console application as a starting point. In this lab, we'll be using the SDK - rather than the RESTful API - but the pattern of interaction is very similar.

1. Open **Visual Studio 2017** from the Start Menu.
1. Click **Open Project/Solution** from the **File > Open** menu.
1. Select the solution file `$/CustomVision.sln` and wait for it to load.
1. Right-click the **Training** project and select **Manage NuGet Packages...**.
1. Click on the **Browse** tab.
1. Enter `Microsoft.Cognitive.CustomVision.Training` in the search box and press **Enter**.
1. Click on the matching result and then click **Install** in the right panel. When prompted, click **OK** and then **I Accept**.

### B) Finish the training application

The first step is to upload training data. In our case, the images are local so will be uploaded directly to the service; however, it's also possible to submit public URLs to training images and the service will automatically retrieve them. Once the images are uploaded, we can start the training process. This is an asynchronous operation so we need to monitor it.

1. Open the **Program.cs** file in the **Training** project.
1. Add the `UploadTrainingData` method to upload the new training images to our project:
    ```cs
    private static void UploadTrainingData(Guid projectId, TrainingApi trainingApi)
    {
        Console.WriteLine("Uploading images");
        foreach (var tagName in new[] {"surface-pro", "surface-studio"})
        {
            var tag = trainingApi.CreateTag(projectId, tagName);

            var images = Directory.GetFiles($@"..\..\Images\{tagName}")
              .Select(f => new ImageFileCreateEntry(Path.GetFileName(f), File.ReadAllBytes(f)))
              .ToList();

            var batch = new ImageFileCreateBatch(images, new List<Guid> { tag.Id });
            trainingApi.CreateImagesFromFiles(projectId, batch);
        }
    }
    ```
1. Add the `TrainClassifier` method to invoke the training operation and wait for it to complete:
    ```cs
    private static void TrainClassifier(Guid projectId, TrainingApi trainingApi)
    {
        Console.WriteLine("Training classifier");

        var iteration = trainingApi.TrainProject(projectId);
        while (iteration.Status == "Training")
        {
            Thread.Sleep(1000);
            iteration = trainingApi.GetIteration(projectId, iteration.Id);
        }

        // Make the newly trained version the default for RESTful API requests
        iteration.IsDefault = true;
        trainingApi.UpdateIteration(projectId, iteration.Id, iteration);

        Console.WriteLine("Training complete. Press any key to exit.");
    }
    ```
1. Add the following code to the `Main` method to retrieve credentials and invoke our new methods:
    ```cs
    var trainingApi = new TrainingApi();
    trainingApi.ApiKey = ConfigurationManager.AppSettings["trainingKey"];

    var projectName = ConfigurationManager.AppSettings["projectName"];
    var project = trainingApi.GetProjects().Single(p => p.Name == projectName);

    UploadTrainingData(project.Id, trainingApi);
    TrainClassifier(project.Id, trainingApi);

    Console.ReadKey();
    ```

### C) Add required configuration settings

In order to interact with the training API, we need the key. This gives us access to all projects under our Custom Vision Service account. To identify the correct project to work with, we need the project name as well.

1. Return to the browser and the [Custom Vision](https://customvision.ai) (customvision.ai) portal.
1. Click the **home** icon in the top left corner of the portal.
1. Click **New Project**.
1. Provide the required values:
    * Name: `customvisionlab-api`
    * Description: `Custom Vision Lab - API`
    * Domains: `General` (i.e. the default)
1. Click **Create Project**.
1. Click on the **settings** icon in the top right corner of portal.
1. Copy the **Training Key** located under the *Subscription Keys* section.
1. Return to **Visual Studio**.
1. Open the **App.config** file in the **Training** project.
1. Find the **trainingKey** app setting and replace `<your_key>` with the key you copied to clipboard.
    ```xml
    <add key="trainingKey" value="<your_key>" />
    ```
1. Find the **projectName** app setting and update it with your project name (`customvisionlab-api`).
    ```xml
    <add key="projectName" value="<your_project>" />
    ```

### D) Run the console application

The application will use the images under `$\Training\Images\{tag}` in the training process. The entire process should only take a few moments given the small size of our data set.

1. In Visual Studio, right-click the **Training** project.
1. Under **Debug**, click **Start new instance**.
1. Wait for the upload and training process to complete.
1. Once you see the **Training complete** message, press **any key** to exit.
1. Return to **Microsoft Edge** and the [Custom Vision](https://customvision.ai) (customvision.ai) portal.
1. Open your project (`customvisionlab-api`) and click the **Performance** tab.
1. Here we can see the classifier's precision and recall.

## Use the prediction API (5 min)

### A) Finish the predication application

Now that the classifier is in place, we can use the prediction endpoint to classify new, unseen data. This can be accomplished with the prediction API.

1. Return to **Visual Studio**.
1. Right-click the **Prediction** project and select **Manage NuGet Packages...**.
1. Click on the **Browse** tab.
1. Enter `Microsoft.Cognitive.CustomVision.Training` in the search box and press **Enter**.
1. Click on the matching result and then click **Install** in the right panel.
    > In our app, the training SDK is also required in order to resolve the project ID from the project name. The alternative would be to add the project ID to configuration and then this dependency can be eliminated.
1. Enter `Microsoft.Cognitive.CustomVision.Prediction` in the search box and press **Enter**.
1. Click on the matching result and then click **Install** in the right panel. When prompted, click **OK** and then **I Accept**.
1. Open **Program.cs** in the **Prediction** project.
1. Add `GetProjectId` method to resolve the project ID from the project name in our config file:
    ```cs
    private static Guid GetProjectId()
    {
        var trainingApi = new TrainingApi();
        trainingApi.ApiKey = ConfigurationManager.AppSettings["trainingKey"];

        var projectName = ConfigurationManager.AppSettings["projectName"];
        var project = trainingApi.GetProjects().Single(p => p.Name == projectName);

        return project.Id;
    }
    ```
1. Add `GetPredictionEndpoint` method to retrieve credentials and prepare the SDK to make predictions:
    ```cs
    private static PredictionEndpoint GetPredictionEndpoint()
    {
        var predictionEndpoint = new PredictionEndpoint();
        predictionEndpoint.ApiKey = ConfigurationManager.AppSettings["predictionKey"];
        return predictionEndpoint;
    }
    ```
1. Add the following code to the `Main` method to invoke the prediction endpoint:
    ```cs
    var projectId = GetProjectId();
    var endpoint = GetPredictionEndpoint();

    Console.WriteLine("Testing images");
    foreach (var tagName in new[] {"surface-pro", "surface-studio"})
    {
        var images = Directory.GetFiles($@"..\..\Images\{tagName}");
        foreach (var image in images)
        {
            var memoryStream = new MemoryStream(File.ReadAllBytes(image));
            var prediction = endpoint.PredictImage(projectId, memoryStream);
            
            var topPrediction = prediction.Predictions.First();
            Console.WriteLine($"{image} predicted to be {topPrediction.Tag} ({topPrediction.Probability:P1})");
        }
    }

    Console.WriteLine("Press any key to exit");
    Console.ReadKey();
    ```

### B) Add required configuration settings

In addition to the information required by the training API to resolve the project ID, a separate prediction API key is required to access the prediction functionality.

1. Open the **App.config** file in the **Prediction** project.
1. Find the **trainingKey** app setting and replace `<your_key>` with the key from the **Training** project **App.config**.
    ```xml
    <add key="trainingKey" value="<your_key>" />
    ```
1. Find the **projectName** app setting and replace `<your_project>` with the project name from the **Training** project **App.config**.
    ```xml
    <add key="projectName" value="<your_project>" />
    ```
1. Return to the browser and the [Custom Vision](https://customvision.ai) (customvision.ai) portal.
1. Click on the **settings** icon in the top right corner of portal.
1. Copy the **Prediction Key** located under the *Subscription Keys* section.
1. Return to **Visual Studio**.
1. Find the **predictionKey** app setting and replace `<your_key>` with the key you copied to clipboard.
    ```xml
    <add key="predictionKey" value="<your_key>" />
    ```

### C) Run the console application

The application will use the images under `$/Prediction/Images/{tag}` and return the most likely predictions for each.

1. In Visual Studio, right-click the **Prediction** project.
1. Under **Debug**, click **Start new instance**.
1. Review the output and ensure the images are correctly classified. The `pro_test.jpg` should be classified as a `surface-pro` and the `studio_test.jpg` should be classified as a `surface-studio`.
1. Once prompted, press **any key** to exit.
1. Return to the browser and the [Custom Vision](https://customvision.ai) (customvision.ai) portal.
1. Open your project (`customvisionlab-api`) and click the **Predictions** tab.
1. As before, we can see the new data collected by the prediction endpoint. If we click on an image, we can manually tag it and add it to our data set.

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

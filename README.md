# PTIAnalyzers
This package allows you to get some insights on sentiment and personality based on a given text. Package requires you to have credentials for Azure Text Analytics and IBM Personality Insights

## Pre- Requirements
In order to be able to use the package, you need to have the following
* Azure Text Analytics. 
  * You can learn how to configure one here: https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-apis-create-account
  * Get the access key and endpoint url https://docs.microsoft.com/en-us/azure/cognitive-services/text-analytics/how-tos/text-analytics-how-to-access-key
* IBM Personality Insights
  * You can learn how to create an account here: https://cloud.ibm.com/docs/services/personality-insights?topic=personality-insights-gettingStarted#gettingStarted
  * Get the username and password https://cloud.ibm.com/docs/services/watson?topic=watson-creating-credentials#creating-credentials
  

## Analyze a given text
* Install the package https://www.nuget.org/packages/PTI.Analyzers/
* Create a new instance of TextAnalyzer and pass it the configured parameters.
* If you want to intercept the http responses received, configure the event handler for OnHttpResponseReceivedEventHandler
* Create a new instance of DataAnalyzer
* Invoke "AnalyzeTextAndGenerateExcelFileAsync". It will return a byte[] corresponding to the generated XLSX file.
* Do whatever you need to do with the generated bytes, such as save it to disk.

               TextAnalyzer textAnalyzer = new TextAnalyzer(new MSTextAnalyticsConfiguration(Configuration.MSTextAnalyticsKey,
                    Configuration.MSTextAnalyticsEndpoint),
                    new IBMWatsonClientConfiguration(Configuration.IBMPersonalityInsightsUsername,
                    Configuration.IBMPersonalityInsightsPassword));
            textAnalyzer.OnHttpResponseReceivedEventHandler += TextAnalyzer_OnHttpResponseReceivedEventHandler;
            DataAnalyzer dataAnalyzer =
                new DataAnalyzer(
                    textAnalyzer);
            var excelFileBytes = dataAnalyzer.AnalyzeTextAndGenerateExcelFileAsync(fileContents).Result;
            File.WriteAllBytes(@"C:\Temp\GeneratedExcelFile.xlsx", excelFileBytes);
            
## Analyze a list of photos and generates temp files and byte[] with the modified image including the faces rectangles
* Requires having credentials for Azure Face API and Computer Vision
** Check Face API here https://docs.microsoft.com/en-us/azure/cognitive-services/face/overview
** Check Computer Vision here https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/home
    PhotoAnalyzer photoAnalyzer = new PhotoAnalyzer(new MSFaceApiConfiguration(
                FACEKEY, FACEENDPOINT),
                new MSComputerVisionApiConfiguration(VISIONKEY,
                VISIONENDPOINT));
            DataAnalyzer dataAnalyzer = new DataAnalyzer(
                photoAnalyzer);
            List<PhotoInformation> lstPhotos = new List<PhotoInformation>();
            string testFile = @"C:\TestFilesDirectory\testFile.jpg";
            lstPhotos.Add(new PhotoInformation()
            {
                Filename = testFile,
                PhotoStream = File.OpenRead(testFile)
            });
            await dataAnalyzer.AnalyzePhotoStream(lstPhotos, Color.Blue, 10, @"C:\Temp\GeneratedFiles");
            foreach (var singlePhoto in lstPhotos)
            {
                var filenameWithoutPath = System.IO.Path.GetFileName(singlePhoto.Filename);
                var newFileFullPath = $@"C:\Temp\ModifiedPhoto{filenameWithoutPath}";
                File.WriteAllBytes(newFileFullPath, singlePhoto.ModifiedPhotoBytes);
            }

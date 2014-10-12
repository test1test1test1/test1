Azure Machine Learning Recommendations – 
Quick start guide
This document depicts how to onboard your service or application to use Azure ML Recommendations. 
Contents
1.	General Overview	2
2.	Limitations	2
3.	Integration	2
3.1.	Authentication	2
3.2.	Service URI	2
3.3.	API Version	3
3.4.	Create a model	3
3.5.	Import catalog data	4
3.6.	Import usage data	5
3.6.1.	Uploading file	5
3.6.2.	Using data acquisition	6
3.7.	Build recommendation model	7
3.8.	Get Build Status	8
3.9.	Get Recommendations	9
3.10.	Update Model	9
4.	Legal	11

 
1.	General Overview
To use Azure ML Recommendations you need to do the following steps:
1.	Create a Model – A model is a container of your usage data, catalog data and the recommendation model.
2.	Import catalog data – This is an optional step. A catalog contains meta-data information on the items. If you do not upload catalog data, the recommendations services will learn about your catalog implicitly from the usage data.
3.	Import usage data – Usage data can be uploaded in one of 2 ways (or both):
a.	By uploading a file that contains the usage data.
b.	By sending Data Acquisition events.
Usually you upload a usage file in order to be able to create an initial recommendation model (bootstrap) and use it until the system gathers enough data using the data acquisition format.
4.	Build a recommendation model – this is an asynchronous operation in which the recommendation system takes all the usage data and creates a recommendation model. This operation can take several minutes or several hours depending on the size of the data and the build configuration parameters. When triggering the build you will get a build id, use it to check when the build process has ended before starting to consume recommendations. 
5.	Recommendations consumption – get recommendations for a specific item or list of items.
All the steps above are done through Azure ML Recommendations API
2.	Limitations
•	Maximum number of models per subscription: 10
•	Maximum number of items that a catalog can hold: 100,000
•	Maximum size of data can be sent in POST (e.g. Import catalog data, import usage data) is 4MB
•	The number of transactions per second for a recommendation model build that is not active is ~2TPS, only recommendation model build that is active can hold up to 20TPS
3.	Integration
3.1.	Authentication
Please follow Data Market guidelines regarding authentication. Data Market supports either Basic or OAuth authentication methods.
3.2.	Service URI 
The service root URIs for each of the Azure ML Recommendations APIs is:
https://api.datamarket.azure.com/amla/recommendations/v1/


The full service URI is expressed using elements of the OData specification.  
3.3.	API Version
Each API call will have at the end query parameter called apiVersion that should be set to 1.0


3.4.	Create a model
Creating a “create model” request:
HTTP Method	URI
POST	<rootURI>/CreateModel?modelName=%27<model_name>%27&apiVersion=%271.0%27

Example:
<rootURI>/CreateModel?modelName=%27MyFirstModel%27&apiVersion=%271.0%27

Parameter Name	Valid values
modelName	Only letters (A-Z, a-z), numbers (0-9), hyphens (-) and underscore (_) are allowed
Max length: 20
apiVersion	1.0


Request Body	NONE

Response:
HTTP Status code	200
OData XML	[XML– need to think how to show it to user]



3.5.	Import catalog data
If you upload several catalog files to the same model with several calls we will insert only the new catalog items. Existing items will remain with the original values.

HTTP Method	URI
POST	<rootURI>/ImportCatalogFile?modelId=%27<modelId>%27&filename=%27<fileName>%27&apiVersion=%271.0%27

Example:
<rootURI>/ImportCatalogFile?modelId=%27a658c626-2baa-43a7-ac98-f6ee26120a12%27&filename=%27catalog10_small.txt%27&apiVersion=%271.0%27

Parameter Name	Valid values
modelId	The unique identifier of the model. 
filename	Textual identifier of the catalog.
Only letters (A-Z, a-z), numbers (0-9), hyphens (-) and underscore (_) are allowed
Max length: 50
apiVersion	1.0


Request body	The catalog data. Format:
<Item Id>,<Item Name>,<Item Category>[,<description>] 

Name	Mandatory	Type 	Description
Item Id	Yes	Alphanumeric, Max Length 50	Unique identifier of an Item
Item Name	Yes	Alphanumeric, Max Length 255	The Item Name
Item Category	Yes	Alphanumeric, Max Length 255	The category to which this item belongs (e.g. Cooking Books, Drama…)
Description	No	Alphanumeric, Max Length 4000	A description of this item
Maximum file size 200MB

Example:
2406e770-769c-4189-89de-1c9283f93a96,Clara Callan,Book
21bf8088-b6c0-4509-870c-e1c7ac78304a,The Forgetting Room: A Fiction (Byzantium Book),Book
3bb5cb44-d143-4bdd-a55c-443964bf4b23,Spadework,Book
552a1940-21e4-4399-82bb-594b46d7ed54,Restraint of Beasts,Book


Response:
HTTP Status code	200
OData XML	[XML– need to think how to show it to user]



3.6.	Import usage data
3.6.1.	 Uploading file
This sections shows how to upload usage data using a file. You can call this API several times with usage data. All usage data will be saved for all calls.
HTTP Method	URI
POST	<rootURI>/ImportUsageFile?modelId=%27<modelId>%27&filename=%27<fileName>%27&apiVersion=%271.0%27

Example:
<rootURI>/ImportUsageFile?modelId=%27a658c626-2baa-43a7-ac98-f6ee26120a12%27&filename=%27ImplicitMatrix10_Guid_small.txt%27&apiVersion=%271.0%27

Parameter Name	Valid values
modelId	The unique identifier of the model. 
filename	Textual identifier of the catalog.
Only letters (A-Z, a-z), numbers (0-9), hyphens (-) and underscore (_) are allowed
Max length: 50
apiVersion	1.0


Request body	The usage data. Format:
<User Id>,<Item Id>[,<Time>,<EventType>]

Name	Mandatory	Type 	Description
User Id	Yes	Alphanumeric	Unique identifier of a User
Item Id	Yes	Alphanumeric, Max Length 50	Unique identifier of an Item
Timestamp	No	Date in format: YYYY/MM/DDTHH:MM:SS (e.g. 2013/06/20T10:00:00)	Time of data
Event	No, if supplied then must also put date	One of the following:
•	Click
•	RecommendationClick
•	AddShopCart
•	RemoveShopCart
•	Purchase	
Maximum file size 200MB

Example:
149452,1b3d95e2-84e4-414c-bb38-be9cf461c347
6360,1b3d95e2-84e4-414c-bb38-be9cf461c347
50321,1b3d95e2-84e4-414c-bb38-be9cf461c347
71285,1b3d95e2-84e4-414c-bb38-be9cf461c347
224450,1b3d95e2-84e4-414c-bb38-be9cf461c347
236645,1b3d95e2-84e4-414c-bb38-be9cf461c347
107951,1b3d95e2-84e4-414c-bb38-be9cf461c347


Response:
HTTP Status code	200
OData XML	[XML– need to think how to show it to user]

3.6.2.	Using data acquisition
This section shows how to send events in real time to Azure ML Recommendations usually from you web site.
HTTP Method	URI
POST	<rootURI>/AddUsageEvent?apiVersion=%271.0%27

Parameter Name	Valid values
apiVersion	1.0


Request body	Event data entry for each event you want to send

Example:
<Event xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ModelId>2779c063-48fb-46c1-bae3-74acddc8c1d1</ModelId>
  <SessionId>11112222</SessionId>
  <EventData>
    <EventData>
      <Name>Click</Name>
      <ItemId>21BF8088-B6C0-4509-870C-E1C7AC78304A</ItemId>
      <ItemName>itemName</ItemName>
      <ItemDescription>item description</ItemDescription>
      <ItemCategory>category</ItemCategory>
    </EventData>
    <EventData>
      <Name>AddShopCart</Name>
      <ItemId>552A1940-21E4-4399-82BB-594B46D7ED54</ItemId>
    </EventData>
  </EventData>
</Event>



Response:
HTTP Status code	200

3.7.	Build recommendation model

HTTP Method	URI
POST	<rootURI>/BuildModel?modelId=%27<modelId>%27&userDescription=%27<description>%27&apiVersion=%271.0%27

Example:
<rootURI>/BuildModel?modelId=%27a658c626-2baa-43a7-ac98-f6ee26120a12%27&userDescription=%27First%20build%27&apiVersion=%271.0%27

Parameter Name	Valid values
modelId	The unique identifier of the model. 
userDescription	Textual identifier of the catalog. Note that if you use spaces you must encode it with %20 instead. See example above.
Max length: 50
apiVersion	1.0


Request body	None


Response:
HTTP Status code	200
OData XML	This is async API. You will get a BuildId, you must use this build Id for calling “Get Build Status” API to know when the build has ended.
Valid build status:
•	Create – model was just created
•	Queued – model build was triggered and it is queued
•	Building – the model is being build
•	Success – the build ended successfully
•	Error – the build ended with a failure

[XML– need to think how to show it to user]


3.8.	Get Build Status
HTTP Method	URI
GET	<rootURI>/ListBuildsForModel?modelId=%27<modelId>%27&apiVersion=%271.0%27

Example of importing a usage:
<rootURI>/ListBuildsForModel?modelId=%279559872f-7a53-4076-a3c7-19d9385c1265%27&apiVersion=%271.0%27

Parameter Name	Valid values
modelId	The unique identifier of the model. 
apiVersion	1.0

Response:
HTTP Status code	200
OData XML	Extract the build status from the XML
Valid build status:
•	Create – model was just created
•	Queued – model build was triggered and it is queued
•	Building – the model is being build
•	Success – the build ended successfully
•	Error – the build ended with a failure

[XML– need to think how to show it to user]



3.9.	Get Recommendations
HTTP Method	URI
GET	<rootURI>/ItemRecommend?modelId=%27<modelId>%27&itemIds=%27<itemId>%27&numberOfResults=<int>&includeMetadata=<bool>&apiVersion=%271.0%27

Example:
<rootURI>/ItemRecommend?modelId=%272779c063-48fb-46c1-bae3-74acddc8c1d1%27&itemIds=%271003%27&numberOfResults=10&includeMetadata=false&apiVersion=%271.0%27

Parameter Name	Valid values
modelId	The unique identifier of the model. 
itemIds	Comma separated list of the items to recommend for
Max length: 200
numberOfResults	The number of required results
includeMetatadata	Future use, put always false. 
apiVersion	1.0
Response:
HTTP Status code	200
OData XML	Extract recommendation results from the XML

[XML– need to think how to show it to user]

3.10.	Update Model
You can update the model description or the active build id.
Active Build Id – Every build for every model has a build id. The active build id is the first successfully build of every new model. Once you have an active build Id and you do additional builds for the same model you need to explicit set it as the default build id if you want to. When you consume recommendations, if you do not specify the build id that you want to use the default one will be sued automatically.
This mechanism enables you once you have a recommendation model in production to build new models and test them before you promote them to production.

HTTP Method	URI
PUT	<rootURI>/UpdateModel?id=%27<modelId>%27&apiVersion=%271.0%27

Example of importing a usage:
<rootURI>/UpdateModel?id=%279559872f-7a53-4076-a3c7-19d9385c1265%27&apiVersion=%271.0%27

Parameter Name	Valid values
id	The unique identifier of the model. 
apiVersion	1.0


Request body	<ModelUpdateParams xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        <Description>New Description</Description>
        <ActiveBuildId>-1</ActiveBuildId>
</ModelUpdateParams>

Note that the xml tags Description and ActiveBuildId are optional, If you do not want to set Description or ActiveBuildId remove the entire tag.

Response:
HTTP Status code	200
OData XML	[XML – need to think how to show it to user]

[Need sample app]
4.	Legal
This document is provided “as-is”. Information and views expressed in this document, including URL and other Internet Web site references, may change without notice. 
Some examples depicted herein are provided for illustration only and are fictitious. No real association or connection is intended or should be inferred. 
This document does not provide you with any legal rights to any intellectual property in any Microsoft product. You may copy and use this document for your internal, reference purposes. 
© 2014 Microsoft. All rights reserved. 

[Appendix – Need to think what to do with below]
Create Model XML Response:
<feed xmlns:base="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/CreateModel" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xmlns="http://www.w3.org/2005/Atom">
  <title type="text" />
  <subtitle type="text">Create A New Model</subtitle>
  <id>https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/CreateModel?modelName='MyFirstModel'&amp;apiVersion='1.0'</id>
  <rights type="text" />
  <updated>2014-10-05T06:35:21Z</updated>
  <link rel="self" href="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/CreateModel?modelName='MyFirstModel'&amp;apiVersion='1.0'" />
  <entry>
    <id>https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/CreateModel?modelName='MyFirstModel'&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1</id>
    <title type="text">CreateANewModelEntity2</title>
    <updated>2014-10-05T06:35:21Z</updated>
    <link rel="self" href="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/CreateModel?modelName='MyFirstModel'&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">a658c626-2baa-43a7-ac98-f6ee26120a12</d:Id>
        <d:Name m:type="Edm.String">MyFirstModel</d:Name>
        <d:Date m:type="Edm.String">10/5/2014 6:35:19 AM</d:Date>
        <d:Status m:type="Edm.String">Created</d:Status>
        <d:HasActiveBuild m:type="Edm.String">false</d:HasActiveBuild>
        <d:BuildId m:type="Edm.String">-1</d:BuildId>
        <d:Mpr m:type="Edm.String">0</d:Mpr>
        <d:UserName m:type="Edm.String">5-4058-ab36-1fe254f05102@dm.com</d:UserName>
        <d:Description m:type="Edm.String"></d:Description>
      </m:properties>
    </content>
  </entry>
</feed>


Import Catalog Data XML Response
<feed xmlns:base="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ImportCatalogFile" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xmlns="http://www.w3.org/2005/Atom">
  <title type="text" />
  <subtitle type="text">Import catalog file</subtitle>
  <id>https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ImportCatalogFile?modelId='a658c626-2baa-43a7-ac98-f6ee26120a12'&amp;filename='catalog10_small.txt'&amp;apiVersion='1.0'</id>
  <rights type="text" />
  <updated>2014-10-05T06:58:04Z</updated>
  <link rel="self" href="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ImportCatalogFile?modelId='a658c626-2baa-43a7-ac98-f6ee26120a12'&amp;filename='catalog10_small.txt'&amp;apiVersion='1.0'" />
  <entry>
    <id>https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ImportCatalogFile?modelId='a658c626-2baa-43a7-ac98-f6ee26120a12'&amp;filename='catalog10_small.txt'&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1</id>
    <title type="text">ImportCatalogFileEntity2</title>
    <updated>2014-10-05T06:58:04Z</updated>
    <link rel="self" href="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ImportCatalogFile?modelId='a658c626-2baa-43a7-ac98-f6ee26120a12'&amp;filename='catalog10_small.txt'&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:LineCount m:type="Edm.String">4</d:LineCount>
        <d:ErrorCount m:type="Edm.String">0</d:ErrorCount>
      </m:properties>
    </content>
  </entry>
</feed>


Import Usage data XML response
<feed xmlns:base="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ImportUsageFile" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xmlns="http://www.w3.org/2005/Atom">
  <title type="text" />
  <subtitle type="text">Add bulk usage data (usage file)</subtitle>
  <id>https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ImportUsageFile?modelId='a658c626-2baa-43a7-ac98-f6ee26120a12'&amp;filename='ImplicitMatrix10_Guid_small.txt'&amp;apiVersion='1.0'</id>
  <rights type="text" />
  <updated>2014-10-05T07:21:44Z</updated>
  <link rel="self" href="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ImportUsageFile?modelId='a658c626-2baa-43a7-ac98-f6ee26120a12'&amp;filename='ImplicitMatrix10_Guid_small.txt'&amp;apiVersion='1.0'" />
  <entry>
    <id>https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ImportUsageFile?modelId='a658c626-2baa-43a7-ac98-f6ee26120a12'&amp;filename='ImplicitMatrix10_Guid_small.txt'&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1</id>
    <title type="text">AddBulkUsageDataUsageFileEntity2</title>
    <updated>2014-10-05T07:21:44Z</updated>
    <link rel="self" href="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ImportUsageFile?modelId='a658c626-2baa-43a7-ac98-f6ee26120a12'&amp;filename='ImplicitMatrix10_Guid_small.txt'&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:LineCount m:type="Edm.String">33</d:LineCount>
        <d:ErrorCount m:type="Edm.String">0</d:ErrorCount>
        <d:FileId m:type="Edm.String">fead7c1c-db01-46c0-872f-65bcda36025d</d:FileId>
      </m:properties>
    </content>
  </entry>
</feed>

Start Build XML response
<feed xmlns:base="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/BuildModel" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xmlns="http://www.w3.org/2005/Atom">
  <title type="text" />
  <subtitle type="text">Build a Model with RequestBody</subtitle>
  <id>https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/BuildModel?modelId='9559872f-7a53-4076-a3c7-19d9385c1265'&amp;userDescription='First build'&amp;apiVersion='1.0'</id>
  <rights type="text" />
  <updated>2014-10-05T08:56:34Z</updated>
  <link rel="self" href="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/BuildModel?modelId='9559872f-7a53-4076-a3c7-19d9385c1265'&amp;userDescription='First%20build'&amp;apiVersion='1.0'" />
  <entry>
    <id>https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/BuildModel?modelId='9559872f-7a53-4076-a3c7-19d9385c1265'&amp;userDescription='First build'&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1</id>
    <title type="text">BuildAModelEntity2</title>
    <updated>2014-10-05T08:56:34Z</updated>
    <link rel="self" href="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/BuildModel?modelId='9559872f-7a53-4076-a3c7-19d9385c1265'&amp;userDescription='First%20build'&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">1000272</d:Id>
        <d:UserName m:type="Edm.String"></d:UserName>
        <d:ModelId m:type="Edm.String">9559872f-7a53-4076-a3c7-19d9385c1265</d:ModelId>
        <d:ModelName m:type="Edm.String">docTest</d:ModelName>
        <d:Type m:type="Edm.String">Recommendation</d:Type>
        <d:CreationTime m:type="Edm.String">2014-10-05T08:56:31.893</d:CreationTime>
        <d:Progress_BuildId m:type="Edm.String">1000272</d:Progress_BuildId>
        <d:Progress_ModelId m:type="Edm.String">9559872f-7a53-4076-a3c7-19d9385c1265</d:Progress_ModelId>
        <d:Progress_UserName m:type="Edm.String">5-4058-ab36-1fe254f05102@dm.com</d:Progress_UserName>
        <d:Progress_IsExecutionStarted m:type="Edm.String">false</d:Progress_IsExecutionStarted>
        <d:Progress_IsExecutionEnded m:type="Edm.String">false</d:Progress_IsExecutionEnded>
        <d:Progress_Percent m:type="Edm.String">0</d:Progress_Percent>
        <d:Progress_StartTime m:type="Edm.String">0001-01-01T00:00:00</d:Progress_StartTime>
        <d:Progress_EndTime m:type="Edm.String">0001-01-01T00:00:00</d:Progress_EndTime>
        <d:Progress_UpdateDateUTC m:type="Edm.String"></d:Progress_UpdateDateUTC>
        <d:Status m:type="Edm.String">Queued</d:Status>
        <d:Key1 m:type="Edm.String">UseFeaturesInModel</d:Key1>
        <d:Value1 m:type="Edm.String">False</d:Value1>
      </m:properties>
    </content>
  </entry>
</feed>

Response for update model
<feed xmlns:base="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/UpdateModel" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xmlns="http://www.w3.org/2005/Atom">
  <title type="text" />
  <subtitle type="text">Update an Existing Model</subtitle>
  <id>https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/UpdateModel?id='9559872f-7a53-4076-a3c7-19d9385c1265'&amp;apiVersion='1.0'</id>
  <rights type="text" />
  <updated>2014-10-05T10:27:17Z</updated>
  <link rel="self" href="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/UpdateModel?id='9559872f-7a53-4076-a3c7-19d9385c1265'&amp;apiVersion='1.0'" />
</feed>


Response for GetBuildStatus
<feed xmlns:base="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ListBuildsForModel" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xmlns="http://www.w3.org/2005/Atom">
  <title type="text" />
  <subtitle type="text">List builds for model</subtitle>
  <id>https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ListBuildsForModel?modelId='9559872f-7a53-4076-a3c7-19d9385c1265'&amp;apiVersion='1.0'</id>
  <rights type="text" />
  <updated>2014-10-05T11:12:17Z</updated>
  <link rel="self" href="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ListBuildsForModel?modelId='9559872f-7a53-4076-a3c7-19d9385c1265'&amp;apiVersion='1.0'" />
  <entry>
    <id>https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ListBuildsForModel?modelId='9559872f-7a53-4076-a3c7-19d9385c1265'&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1</id>
    <title type="text">ListBuildsForModelEntity</title>
    <updated>2014-10-05T11:12:17Z</updated>
    <link rel="self" href="https://api.sqlazureservices-test2.com/Data.ashx/amla/recommendations-preview/v2/ListBuildsForModel?modelId='9559872f-7a53-4076-a3c7-19d9385c1265'&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:BuildId m:type="Edm.String">1000272</d:BuildId>
        <d:Status m:type="Edm.String">Error</d:Status>
        <d:UserDescription m:type="Edm.String"></d:UserDescription>
        <d:StartTime m:type="Edm.String">2014-10-05T08:56:36</d:StartTime>
        <d:EndTime m:type="Edm.String">2014-10-05T08:58:37</d:EndTime>
        <d:Duration m:type="Edm.String">00:02:01</d:Duration>
      </m:properties>
    </content>
  </entry>
</feed>

Recommendation response xml:
<feed xmlns:base="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xmlns="http://www.w3.org/2005/Atom">
  <title type="text" />
  <subtitle type="text">Get Recommendation</subtitle>
  <id>https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'</id>
  <rights type="text" />
  <updated>2014-10-05T12:28:48Z</updated>
  <link rel="self" href="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'" />
  <entry>
    <id>https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1</id>
    <title type="text">GetRecommendationEntity</title>
    <updated>2014-10-05T12:28:48Z</updated>
    <link rel="self" href="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=0&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">159</d:Id>
        <d:Name m:type="Edm.String">159</d:Name>
        <d:Rating m:type="Edm.Double">0.543343480387708</d:Rating>
        <d:Reasoning m:type="Edm.String">People who like '1003' also like '159'</d:Reasoning>
      </m:properties>
    </content>
  </entry>
  <entry>
    <id>https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=1&amp;$top=1</id>
    <title type="text">GetRecommendationEntity</title>
    <updated>2014-10-05T12:28:48Z</updated>
    <link rel="self" href="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=1&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">52</d:Id>
        <d:Name m:type="Edm.String">52</d:Name>
        <d:Rating m:type="Edm.Double">0.539588900748721</d:Rating>
        <d:Reasoning m:type="Edm.String">People who like '1003' also like '52'</d:Reasoning>
      </m:properties>
    </content>
  </entry>
  <entry>
    <id>https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=2&amp;$top=1</id>
    <title type="text">GetRecommendationEntity</title>
    <updated>2014-10-05T12:28:48Z</updated>
    <link rel="self" href="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=2&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">35</d:Id>
        <d:Name m:type="Edm.String">35</d:Name>
        <d:Rating m:type="Edm.Double">0.53842946443853</d:Rating>
        <d:Reasoning m:type="Edm.String">People who like '1003' also like '35'</d:Reasoning>
      </m:properties>
    </content>
  </entry>
  <entry>
    <id>https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=3&amp;$top=1</id>
    <title type="text">GetRecommendationEntity</title>
    <updated>2014-10-05T12:28:48Z</updated>
    <link rel="self" href="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=3&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">124</d:Id>
        <d:Name m:type="Edm.String">124</d:Name>
        <d:Rating m:type="Edm.Double">0.536712832792886</d:Rating>
        <d:Reasoning m:type="Edm.String">People who like '1003' also like '124'</d:Reasoning>
      </m:properties>
    </content>
  </entry>
  <entry>
    <id>https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=4&amp;$top=1</id>
    <title type="text">GetRecommendationEntity</title>
    <updated>2014-10-05T12:28:48Z</updated>
    <link rel="self" href="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=4&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">120</d:Id>
        <d:Name m:type="Edm.String">120</d:Name>
        <d:Rating m:type="Edm.Double">0.533673023762878</d:Rating>
        <d:Reasoning m:type="Edm.String">People who like '1003' also like '120'</d:Reasoning>
      </m:properties>
    </content>
  </entry>
  <entry>
    <id>https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=5&amp;$top=1</id>
    <title type="text">GetRecommendationEntity</title>
    <updated>2014-10-05T12:28:48Z</updated>
    <link rel="self" href="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=5&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">96</d:Id>
        <d:Name m:type="Edm.String">96</d:Name>
        <d:Rating m:type="Edm.Double">0.532544826370521</d:Rating>
        <d:Reasoning m:type="Edm.String">People who like '1003' also like '96'</d:Reasoning>
      </m:properties>
    </content>
  </entry>
  <entry>
    <id>https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=6&amp;$top=1</id>
    <title type="text">GetRecommendationEntity</title>
    <updated>2014-10-05T12:28:48Z</updated>
    <link rel="self" href="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=6&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">69</d:Id>
        <d:Name m:type="Edm.String">69</d:Name>
        <d:Rating m:type="Edm.Double">0.531678607847759</d:Rating>
        <d:Reasoning m:type="Edm.String">People who like '1003' also like '69'</d:Reasoning>
      </m:properties>
    </content>
  </entry>
  <entry>
    <id>https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=7&amp;$top=1</id>
    <title type="text">GetRecommendationEntity</title>
    <updated>2014-10-05T12:28:48Z</updated>
    <link rel="self" href="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=7&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">172</d:Id>
        <d:Name m:type="Edm.String">172</d:Name>
        <d:Rating m:type="Edm.Double">0.530957821375951</d:Rating>
        <d:Reasoning m:type="Edm.String">People who like '1003' also like '172'</d:Reasoning>
      </m:properties>
    </content>
  </entry>
  <entry>
    <id>https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=8&amp;$top=1</id>
    <title type="text">GetRecommendationEntity</title>
    <updated>2014-10-05T12:28:48Z</updated>
    <link rel="self" href="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=8&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">155</d:Id>
        <d:Name m:type="Edm.String">155</d:Name>
        <d:Rating m:type="Edm.Double">0.529093541481333</d:Rating>
        <d:Reasoning m:type="Edm.String">People who like '1003' also like '155'</d:Reasoning>
      </m:properties>
    </content>
  </entry>
  <entry>
    <id>https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=9&amp;$top=1</id>
    <title type="text">GetRecommendationEntity</title>
    <updated>2014-10-05T12:28:48Z</updated>
    <link rel="self" href="https://api.datamarket.azure.com/Data.ashx/amla/recommendations-preview/v1/ItemRecommend?modelId='2779c063-48fb-46c1-bae3-74acddc8c1d1'&amp;itemIds='1003'&amp;numberOfResults=10&amp;includeMetadata=false&amp;apiVersion='1.0'&amp;$skip=9&amp;$top=1" />
    <content type="application/xml">
      <m:properties>
        <d:Id m:type="Edm.String">32</d:Id>
        <d:Name m:type="Edm.String">32</d:Name>
        <d:Rating m:type="Edm.Double">0.528917978168322</d:Rating>
        <d:Reasoning m:type="Edm.String">People who like '1003' also like '32'</d:Reasoning>
      </m:properties>
    </content>
  </entry>
</feed>


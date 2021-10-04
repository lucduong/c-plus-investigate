# C+ - Table of contents
1. [Call many request](#call_many_request)
2. [Request many data not use](#request_many_data_not_use)
3. [Cold start, Warm Start](#cold_start)
    1. [Large Dependencies](#dependencies)
    2. [Middleware, validation](#middleware)
4. [Use async illegal](#use_async_illegal)
5. [No Cacher](#no_cacher)
6. [Event Driven](#event_driven)
7. [Client Performance](#client_performance)

# Content

## 1. Call many request <a name="call_many_request"></a>

  #### 1. Root cause (Code Logic)

    - One click call many request from server. Normal case all of them same name functions. Other case difference functions.
  
  #### 2. Solutions

    - Should be one click call one function (normal case). All request should be resolved from server before return to client.

  #### 3. Example

     - Below function count some thing for every projects. 

     - We should be combine to 1 function, send all project Id and check all in server and then return all to client.

![N|Solid](/img/c02.png)

## 2. Request many data not use <a name="request_many_data_not_use"></a>

  #### 1. Root cause (Code Logic)

    -  Normal case: one function return many data not use (redundancy).
  
  #### 2. Solutions

    - Should be return enough data that is client can use  

  #### 3. Example

    - Below function update status. 

    - In case, Just return data : true or false or some data that is client need check or use.

    - If data is normal, constant when user logged system. We can store to cacher or local storage
  
![N|Solid](/img/c03.png)

## 3. Cold start, Warm Start <a name="cold_start"></a>

### 3.1 Large Dependencies <a name="dependencies"></a>

  #### 1. Root cause (Code Logic)

    - This is reason relate to size of source code that be dowloaded when started container
    -  Every function use the same external dependencies  
  
  #### 2. Solutions

    - Cross-check for dev packages each lambda function. Just use packages that is required.

    - If packages already support by Node or some where (Ex lambda runtime). No need install in dependency

    - Only install scope function in full package (Ex: use "lodash.find" not install full lodash package for all) 

    - Remove unnecessary packages

    - Use --production flag -> This should be a common step as well, however, It doesn’t hurt to repeat it. 

    Always remember to use the --production when running npm on production or in your CI/CD script to not install dev dependencies.

    - Reuse available Lambda runtime packages -> Just like aws-sdk, there are other packages such as uuid and dotenv that are already available in the lambda runtime which you can reuse.

    - Move out large non-js files -> Consider hosting large files such as images, or JSON on a private CDN (most likely S3) and read it from there.
    This will cause a trade-off in speed but it could be worth it for you depending on the design of your app.

    - Merge serverless handlers

  #### 3. Example

    - The aws-sdk package alone is over 60MB (uncompressed). This is huge! 
      
    - This was almost everything about our app size issue, our misfortune and also our miracle.

    - The good news is that the aws-sdk comes pre-installed in your Lambda runtime, so you don't need to install it again. Only set it as a dev dependency.
- 
![N|Solid](/img/c04.png)

![N|Solid](/img/c05.png)

![N|Solid](/img/c06.png)

## 3.2 Middleware, validation <a name="middleware"></a>

### 1. Root case (Struts Architecture)

 - We have AWS WAF support for validator. So we no need create new layer Authorizer or Validator in Lambda. Authorizer
  
![N|Solid](/img/c07.png)

![N|Solid](/img/c08.png)

![N|Solid](/img/c09.png)

### 3.3 Schema validator

  #### 1. Root cause (Code Logic)

    - The JOI validator is one of schema validator, therefore we shouldn't define the schema during run-time, this will increase the invoke time.

  #### 2. Solution

    - Define JOI validator schema in the initialization step.

![N|Solid](/img/c10.png)

## 4. Use async illegal <a name="use_async_illegal"></a>

#### 1. Root cause (Code Logic)
    -  When we use await inside the iterator operation, the system will await the process completed before moving to the next step.
    -> This is synchronization, not asynchrozation

#### 2. Solutions

     - Don't use async in case no need get / return data

#### 3. Example

![N|Solid](/img/c11.png)
 

## 5. No Cacher <a name="no_cacher"></a>

#### 1. Root cause

    - Client makes frequent API calls to retrieve the data, therefore the function will access the database frequently.
    - In many cases, we don't need to access the database because of no changes (E.g: Historical messages).

#### 2. Solutions

    - Client makes frequent API calls to retrieve static data can benefit from a caching layer.
    - This can reduce the function’s latency, particularly for synchronous requests, as the data is retrieved from the cache instead of an external service.
    - The cache can also reduce costs by reducing the number of calls to downstream services.
  
3. Example
   
![N|Solid](/img/c12.png)
 
![N|Solid](/img/c13.png)

## 6. Event Driven <a name="event_driven"></a>

#### 1. Root cause

     - To begin with, in an event-driven microservice architecture, services communicate each-other via event messages. Thus, the main benefits of event-driven systems are asynchronous behavior and loosely coupled structures.
     - For example, instead of requesting data when needed, apps consume them via events before the need

#### 2. Solutions

      - Use an event-driven, eventually consistent approach. Each service publishes an event whenever it update its data.
      Other service subscribe to events. When an event is received, a service updates its data.

#### 3. Example

    - After create new order for customer. System will be send email to customer.
    In this case, we just push event to Email service send mail. Main function create order don't care function email finished or not.

## 7. Client Performance <a name="client_performance"></a>

#### 1. Root cause

#### 2. Solutions

#### 3. Example
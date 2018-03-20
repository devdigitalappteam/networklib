# NetworkLib

[![N|Solid](http://www.devdigital.com/themes/devdigital2015/images/logo.png)](http://devdigital.com/)

A Generic Networking Library that handle all DevDigital Webservice request/response.

## About Networking Library
 - Networking Library is a powerful library for doing any type of networking in Android applications which is made on top of Fast Android Networking and HttpURLConnection.
 - Networking Library takes care of each and everything. So you don't have to do anything, just make request and listen for the response.

## Why use Networking Library?
1. Support GET, POST, PUT and DELETE request
2. Support JSON, plain Text and key-value param as POST request body and key-value param as GET request query param. 
3. Currently Support AndroidNetworking and HttpURLConnection client for http request. 
    -  In future if require support for any other client just need to implement NetworkStack interface. It will not affect rest of the functionality.
	- Having issue with any HTTP clinet just implement Network Stack of your and it will work.
4. Common Response handler. No need to handle response separtley for each webservice call.
5. Common Progress dialog display. As soon as web request execute this lib will display progress dialog and on response it will close the dialog.
6. generic Json parser. currently using gson but in future if require to use any other library need to change in only 1 class and 2 methods.
7. Intercept web request using stetho if AndroidNetworking http stack is used.
8. Remove Networking boiler plate codes
9. Thought application common response handler and well defined networking structure.


## Library Used

NetworkLib uses a number of open source projects to work properly:

* [Fast Android Networking] - Networking Library that also supports HTTP/2
* [stetho] - To intercept network call.

### Requirements
- Networking Library can be included in any Android application.
- Networking Library supports Android 4.2 and later.	
- Networking Library supports for response format as below:
        
        {
        	"message": "success/failure message",
        	"method": "webservice url method name",
        	"status": "success/failure",
        	"values": {
        		"entityName": "entityData in JSONObject or JSONArray format"
        	}
        }
			
		
    If json format is other than this, then network lib will fail to parse json and show error message.	If json has any fields extra then it will be ignored.	



- Add this in your build.gradle
            	
        implementation 'com.devdigital:networklib:1.6.4'
        

- Then initialize it in onCreate() Method of application class :
            	
        1. Then initialize it in onCreate() Method of application class :

		//If Fast Android Networking Stack is used.
		NetworkController.initAndroidNetworking(this, BuildConfig.DEBUG);

		//Set WebService Handler class to handle WebService Response, Where WebResponseHandler is custom Response handler class in app module.
		//WebResponseHandler must extends {@link BaseWebResponseHandler} class
		NetworkController.setWebResponseHandler(WebResponseHandler.class);
	
	
- Extra Classes needs to be added:
            	
        1. WebResponseHandler
		//WebResponseHandler is custom Response handler class in app module.     WebResponseHandler must extends {@link BaseWebResponseHandler} class

        2. Create Entity class
		NetworkLib already have BaseEntity class which will parse common json attributes, but to parse entity we need to extends BaseEntity class.
		e.g.
			public class AuthorizeEntity extends BaseEntity {
				private AuthorizeModel values;
				public void setValues(AuthorizeModel values){
					this.values = values;
				}
				public AuthorizeModel getValues(){
					return values;
				}
			}
			
		3. Next we need to create Model class, which contains all the field of entity class.
			public class AuthorizeModel {
				private String authorizationCode;
				public void setAuthorizationCode(String authorizationCode){
					this.authorizationCode = authorizationCode;
				}
				public String getAuthorizationCode(){
					return authorizationCode;
				}
			}	

		For json:
			{
				"message": "You are successfully authenticated to use Servyc Tek Solutions resources.",
				"method": "authorize",
				"status": "success",
				"values": {
					"authorizationCode": "fbd006e9be09bc101317199a4fd1324f"
				}
			}
			
			"values" json field represent by AuthorizeEntity and "values" json fields attributes represent by AuthorizeModel class.
         

	

### Making a POST Request Async 

	    Map<String, Object> headers = new HashMap<>();
        headers.put(Header.CONTENT_TYPE, "application/json");
        headers.put(Header.APPLICATION_KEY, "2Xbs2GyPQ2UXPvE3u5CRGpdQASSeA4yz");
        headers.put(Header.APPLICATION_SECRET, "gFefAewjc7BPaHthxLSNWjKhyRaNQUdp");

        Map<String, Object> params = new HashMap<>();
        params.put(Body.APP_VERSION, "1.0");
        params.put(Body.DEVICE_ID, "1234567890");
        params.put(Body.DEVICE_TYPE, "Android");

        String url = "http://servyctek.devdigdev.com/portal/api/v1/authorize";
        WebRequestExecutor requestExecutor = new WebRequestBuilder(this, url)
                .setHeader(headers)
                .setRequestJSON(params)
                .setRequestMethod(WebRequestBuilder.RequestMethod.POST)
                .setEntityClass(AuthorizeEntity.class)
                .setShowProgressDialog(true)
                .setShowErrorDialog(true)
                .setHandleError(true)
                .setResponseListener(new IWebResponseListener() {
                    @Override
                    public void onResponse(ResponseStatus status, Object result) {
                        Log.e(TAG, result.toString());
                        if(status==ResponseStatus.SUCCESS){
							//result is instance of AuthorizeEntity 
                            mContentText.setText(result.toString());
                        }else if(status==ResponseStatus.SERVER_ERROR){
							//result is instance of BaseEntity 
                            mContentText.setText("SERVER ERROR : "+result.toString());
                        }else {
							//result is instance of NetworkResponse 
                            mContentText.setText("HTTP ERROR"+result!=null?result.toString():"");
                        }
                    }
                })
                .build();

		//For Async Call		
        requestExecutor.getAsync();
		
		
### Making a POST Request Sync 
	
        Map<String, Object> headers = new HashMap<>();
        headers.put(Header.CONTENT_TYPE, "application/json");
        headers.put(Header.APPLICATION_KEY, "2Xbs2GyPQ2UXPvE3u5CRGpdQASSeA4yz");
        headers.put(Header.APPLICATION_SECRET, "gFefAewjc7BPaHthxLSNWjKhyRaNQUdp");

        Map<String, Object> params = new HashMap<>();
        params.put(Body.APP_VERSION, "1.0");
        params.put(Body.DEVICE_ID, "1234567890");
        params.put(Body.DEVICE_TYPE, "Android");

        String url = "http://servyctek.devdigdev.com/portal/api/v1/authorize";
        WebRequestExecutor requestExecutor = new WebRequestBuilder(this, url)
                .setHeader(headers)
                .setRequestJSON(params)
                .setRequestMethod(WebRequestBuilder.RequestMethod.POST)
                .setEntityClass(AuthorizeEntity.class)
                .setHandleError(true)
                .build();

		//For Sync Call		
         try {
            AuthorizeEntity entity = (AuthorizeEntity) requestExecutor.get();
            Log.e(TAG, entity.toString());
        }catch (NetworkException e){
            e.printStackTrace();
        }		

### Making GET request Async 

    String url = "http://providencews-dev.devdigdev.com/api/aboutList";
    WebRequestExecutor requestExecutor = new WebRequestBuilder(this, url)
                .setRequestMethod(WebRequestBuilder.RequestMethod.GET)
                .setEntityClass(AuthorizeEntity.class)
                .setShowErrorDialog(true)
                .setHandleError(true)
                .setShowProgressDialog(true)
                .setResponseListener(new IWebResponseListener() {
                    @Override
                    public void onResponse(ResponseStatus status, Object result) {
                        Log.e(TAG, result.toString());
                        if(status==ResponseStatus.SUCCESS){
							//result is instance of AuthorizeEntity 
                            mContentText.setText(result.toString());
                        }else if(status==ResponseStatus.SERVER_ERROR){
							//result is instance of BaseEntity 
                            mContentText.setText("SERVER ERROR : "+result.toString());
                        }else {
							//result is instance of NetworkResponse 
                            mContentText.setText("HTTP ERROR"+result!=null?result.toString():"");
                        }
                    }
                })
                .build();

        requestExecutor.getAsync();
		
		
### Making a PUT Request Async 

    String url = "http://providencews-dev.devdigdev.com/api/aboutList";
    WebRequestExecutor requestExecutor = new WebRequestBuilder(this, url)
                .setRequestMethod(WebRequestBuilder.RequestMethod.PUT)
                .setEntityClass(AuthorizeEntity.class)
                .setShowErrorDialog(true)
                .setHandleError(true)
                .setShowProgressDialog(true)
                .setResponseListener(new IWebResponseListener() {
                    @Override
                    public void onResponse(ResponseStatus status, Object result) {
                        Log.e(TAG, result.toString());
                        if(status==ResponseStatus.SUCCESS){
                            mContentText.setText(result.toString());
                        }else if(status==ResponseStatus.SERVER_ERROR){
                            mContentText.setText("SERVER ERROR : "+result.toString());
                        }else {
                            mContentText.setText("HTTP ERROR"+result!=null?result.toString():"");
                        }
                    }
                })
                .build();

        requestExecutor.getAsync();
		
		
### Making a DELETE Request Async 
	
      String url = "http://providencews-dev.devdigdev.com/api/aboutList";
      WebRequestExecutor requestExecutor = new WebRequestBuilder(this, url)
              .setRequestMethod(WebRequestBuilder.RequestMethod.DELETE)
              .setEntityClass(AuthorizeEntity.class)
              .setShowErrorDialog(true)
              .setHandleError(true)
              .setShowProgressDialog(true)
              .setResponseListener(new IWebResponseListener() {
                  @Override
                  public void onResponse(ResponseStatus status, Object result) {
                      Log.e(TAG, result.toString());
                      if(status==ResponseStatus.SUCCESS){
                          mContentText.setText(result.toString());
                      }else if(status==ResponseStatus.SERVER_ERROR){
                          mContentText.setText("SERVER ERROR : "+result.toString());
                      }else {
                          mContentText.setText("HTTP ERROR"+result!=null?result.toString():"");
                      }
                  }
              })
              .build();

      requestExecutor.getAsync();



   [Fast Android Networking]: <https://github.com/amitshekhariitbhu/Fast-Android-Networking>
   [stetho]: <https://github.com/facebook/stetho>

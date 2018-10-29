# HyperVerge Vietnam Documents - API Documentation

## Overview

This documentation describes Vietnam Docs API v1.1. If you have any queries please contact support. The postman collection can be found [here](https://www.getpostman.com/collections/97da26739e6a67557e88).

## Contents
- [HyperVerge Vietnam Documents - API Documentation](#hyperverge-vietnam-documents-api-documentation)
	- [Overview](#overview)
	- [Contents](#contents)
	- [Schema](#schema)
	- [Parameters](#parameters)
	- [Root Endpoint](#root-endpoint)
	- [Authentication](#authentication)
	- [Media Types](#media-types)
	- [Supported APIs](#supported-apis)
	- [Supported Document Types](#supported-document-types)
		- [Response Structure for Each Type](#response-structure-for-each-type)
	- [Confidence Score for Prediction](#confidence-score-for-prediction)
	- [Data logging and clientId](#data-logging-and-clientid)


## Schema

We recommend using HTTPS for all API access. All data is received as JSON, and all image uploads are to be performed as form-data (POST request). Incase of a pdf file input, the key name has to be `pdf` and in all other cases, the key name for the image could be anything apart from `pdf`.

## Parameters
All optional and compulsory parameters are passed as part of the request body.

## Root Endpoint
A `GET` request can be issued to the root endpoint to check for successful connection :

	 curl https://apac.docs.hyperverge.co/v1.1

The `plain/text` reponse of `"AoK!"` should be received.

## Authentication

Currently, a simple appId, appKey combination is passed in the request header. The appId and appKey are provided on request by the HyperVerge team. If you would like to try the API, please reach out to contact@hyperverge.co

	curl -X POST http://apac.docs.hyperverge.co/v1.1/readNID \
	  -H 'appid: xxx' \
	  -H 'appkey: yyy' \
	  -H 'content-type: multipart/form-data;' \
	  -F 'image=@abc.png'


On failed attempt with invalid credentials or unauthorized access the following error message should be received :

	{
	  "status": "failure",
	  "statusCode": "401",
	  "error": {
	    "developerMessage": "unauthorized",
	  }
	}

Please do not expose the appid and appkey on browser applications. In case of a browser application, set up the API calls from the server side.

## Media Types

Currently, `jpeg, png and tiff` images and `pdf` are supported by the HyperVerge Docs image extraction APIs.

## Supported APIs

Can be used to extract information from any or one of the supported documents depending on the endpoint.

* **NID**
	*  **URL**
		* /readNID : used for any of the supported Vietnam National ID documents
	* **Method:** `POST`
	* **Header**
		- content-type : 'formdata'
		- appid
		- appkey
	* **Request Body**
		- 1-2 images or pdfs
	* **Success Response:**
	    * **Code:** 200 <br />
	    * Incase of a properly made request, the response would follow schema.


			```
			{
				"status" : "success",
				"statusCode" : "200",
				"result" : <resultObject>
			}
			```

		    The `resultObject` has the following Schema :

	        ```
			[{
				"details" : {
					"field-1" : {
					    "value" : "extracted-value-1",
					    "conf" : <float-value>,
					    "to-be-reviewed" : "yes/no"
					},
					"field-2" : {
					    "value" : "extracted-value-2",
					    "conf" : <float-value>,
					    "to-be-reviewed" : "yes/no"
					},
					"field-3" : {
					    "value" : "extracted-value-3",
					    "conf" : <float-value>,
					    "to-be-reviewed" : "yes/no"
					},
					..
				},
				"type" : "id_type"
			}]
			```
        
	* **Fraud Check:**
		We check two types of fraud:
		- **Black and White** Check if the image is Black and White. True if `isBlackWhite` is `yes`.
		- **Province to National ID Mapping** Fixed mapping exists between the ID number and the province. We check if this holds true. If the ID Number does not match the province, `provinceMismatch` is `yes`.
		
		The response will have key `fraudCheck` inside the key `details`: 
		```json
		[{
			"details" : {
				"fraudCheck": {
				    "isBlackWhite": "yes/no",
				    "provinceMismatch": "yes/no"
				},
				"field-1" : {
				    "value" : "extracted-value-1",
				    "conf" : <float-value>,
				    "to-be-reviewed" : "yes/no"
				},
				"field-2" : {
				    "value" : "extracted-value-2",
				    "conf" : <float-value>,
				    "to-be-reviewed" : "yes/no"
				},
				"field-3" : {
				    "value" : "extracted-value-3",
				    "conf" : <float-value>,
				    "to-be-reviewed" : "yes/no"
				},
				..
			},
			"type" : "id_type"	
		}]
		```
		`fraudCheck` key will be present only if `id_type` is `id_front` or `id_new_front`
		
	* **Error Response:**
		There are 3 types of request errors and `HTTP Status Code 400` is returned in all 3 cases:
		- No Image input

			```       
	      	{
	        	"status": "failure",
	        	"statusCode": "400",
	        	"error": "API call requires atlest one input image"
	      	}
			```

		- More than 2 image input

			```       
	      	{
	        	"status": "failure",
	        	"statusCode": "400",
	        	"error": "API call handles only upto 2 images"
	      	}
			```       

		- Larger than allowed image input

			```       
	      	{
	        	"status": "failure",
	        	"statusCode": "400",
	        	"error": "image size cannot be greater than 6MB"
	      	}
			```       

		All error messages follow the same syntax with the statusCode and status also being a part of the response body, and `string` error message with the description of the error.

		**Server Errors**
		We try our best to avoid these errors, but if by chance they do occur the response code will be 5xx.

	* **Sample Calls:**

		- readNID
			- Input having 2 images

			 	```
			    curl -X POST https://apac.docs.hyperverge.co/v1.1/readNID \
					  -H 'appid: xxx' \
					  -H 'appkey: yyyy' \
					  -H 'content-type: multipart/form-data;\
					  -F 'image1=@image1_path.png'\
					  -F 'image2=@image2_path.png'
				```

			- Input having 2 pdfs

				```
			    curl -X POST https://apac.docs.hyperverge.co/v1.1/readNID \
					  -H 'appid: xxx' \
					  -H 'appkey: yyyy' \
					  -H 'content-type: multipart/form-data;\
					  -F 'image1=@image1_path.pdf'\
					  -F 'image2=@image2_path.pdf'
				```

			- Input having 1 image and 1 pdf

				```	   
			   curl -X POST https://apac.docs.hyperverge.co/v1.1/readNID \
					  -H 'appid: xxx' \
					  -H 'appkey: yyyy' \
					  -H 'content-type: multipart/form-data;\
					  -F 'image1=@image1_path.jpg'\
					  -F 'image2=@image2_path.pdf'
				```

* **MRC**
	*  **URL**
		* /readMRC : used for any of the supported Vietnam Motor Registration Certificates
	* **Method:** `POST`
	* **Header**
		- content-type : 'formdata'
		- appid
		- appkey
	* **Request Body**
		- image1: \<type: Image/PDF file\>
		- image2: \<type: Image/PDF file\> (optional)
		- dateFirstRegistration: \<type: Date String in DD/MM/YYYY format, description: Date of First Registration Input\>
		- dateCurrentRegistration: \<type: Date String in DD/MM/YYYY  format, description: Date of Current Registration\>
		- name: \<type: String, description: Full Name of the Owner\>
	* **Success Response:**
	    * **Code:** 200 <br />
	    * Incase of a properly made request, the response would follow schema.

			```
			{
				"status" : "success",
				"statusCode" : "200",
				"result" : <resultObject>
			}
			```

		    The `resultObject` has the following Schema :

	        ```
			[{
				"details" : {
					"field-1" : {
					    "value" : "extracted-value-1",
					    "conf" : <float-value>,
					    "to-be-reviewed" : "yes/no"
					},
					"field-2" : {
					    "value" : "extracted-value-2",
					    "conf" : <float-value>,
					    "to-be-reviewed" : "yes/no"
					},
					"field-3" : {
					    "value" : "extracted-value-3",
					    "conf" : <float-value>,
					    "to-be-reviewed" : "yes/no"
					},
					..,
					"verified": {
						"name": true/false,
						"month-first-registration": true/false,
						"year-first-registration": true/false,
						"month-current-registration": true/false,
						"year-current-registration": true/false
             		}
				},
				"type" : "id_type"
			}]
			```

	* **Error Response:**
		There are 3 types of request errors and `HTTP Status Code 400` is returned in all 3 cases:
		- No Image input

			```       
			{
				"status": "failure",
				"statusCode": "400",
				"error": "API call requires atlest one input image"
			}
			```

		- More than 2 image input

			```
			{
				"status": "failure",
				"statusCode": "400",
				"error": "API call handles only upto 2 images"
			}
			```

		- Larger than allowed image input

			```
			{
				"status": "failure",
				"statusCode": "400",
				"error": "image size cannot be greater than 6MB"
			}
			```

		All error messages follow the same syntax with the statusCode and status also being a part of the response body, and `string` error message with the description of the error.

		**Server Errors**
		We try our best to avoid these errors, but if by chance they do occur the response code will be 5xx.


	* **Sample Calls:**

		 - readMRC

			    curl -X POST https://apac.docs.hyperverge.co/v1.1/readMRC \
					  -H 'appid: xxx' \
					  -H 'appkey: yyyy' \
					  -H 'content-type: multipart/form-data;' \
					  -F 'image1=@image_or_pdf_path1.png' \
					  -F 'image2=@image_or_pdf_path2.png' \
					  -F 'name=Name' \
					  -F 'dateFirstRegistration=12/10/2015' \
					  -F 'dateCurrentRegistration=12/10/2016' \

* **DL**
	*  **URL**
		* /readDL : used for any of the supported Vietnam Driver's License
	* **Method:** `POST`
	* **Header**
		- content-type : 'formdata'
		- appid
		- appkey
	* **Request Body**
		- image1: \<type: Image/PDF file\>
	* **Success Response:**
	    * **Code:** 200 <br />
	    * Incase of a properly made request, the response would follow schema.

			```
			{
				"status" : "success",
				"statusCode" : "200",
				"result" : <resultObject>
			}
			```

		    The `resultObject` has the following Schema :

	        ```
			[{
				"details" : {
					"field-1" : {
					    "value" : "extracted-value-1",
					    "conf" : <float-value>,
					    "to-be-reviewed" : "yes/no"
					},
					"field-2" : {
					    "value" : "extracted-value-2",
					    "conf" : <float-value>,
					    "to-be-reviewed" : "yes/no"
					},
					"field-3" : {
					    "value" : "extracted-value-3",
					    "conf" : <float-value>,
					    "to-be-reviewed" : "yes/no"
					},
					..
				},
				"type" : "id_type"
			}]
			```

	* **Error Response:**
		There are 3 types of request errors and `HTTP Status Code 400` is returned in all 3 cases:
		- No Image input

			```       
			{
				"status": "failure",
				"statusCode": "400",
				"error": "API call requires atlest one input image"
			}
			```

		- More than 2 image input

			```
			{
				"status": "failure",
				"statusCode": "400",
				"error": "API call handles only upto 2 images"
			}
			```

		- Larger than allowed image input

			```
			{
				"status": "failure",
				"statusCode": "400",
				"error": "image size cannot be greater than 6MB"
			}
			```

		All error messages follow the same syntax with the statusCode and status also being a part of the response body, and `string` error message with the description of the error.

		**Server Errors**
		We try our best to avoid these errors, but if by chance they do occur the response code will be 5xx.


	* **Sample Calls:**

		 - readDL

			    curl -X POST https://apac.docs.hyperverge.co/v1.1/readDL \
					  -H 'appid: xxx' \
					  -H 'appkey: yyyy' \
					  -H 'content-type: multipart/form-data;' \
					  -F 'image1=@image_or_pdf_path1.png'


* **EVN**
	*  **URL**
		* /verifyEVN : used for any of the supported Vietnam Electricity Bills
	* **Method:** `POST`
	* **Header**
		- content-type : 'formdata'
		- appid
		- appkey
	* **Request Body**
		- image: \<type: Image/PDF file\>
		- address: \<type: String, description: Address of the Customer\>
		- amount: \<type: String, description: Total Amount after VAT\>
		- evnId: \<type: String, description: EVN ID of the Customer\>
		- name: \<type: String, description: Full Name of the Customer\>
		- fromDate: \<type: Date String in DD/MM/YYYY or DD/MM format, description: From Date of the EVN Bill\>
		- toDate: \<type: Date String in DD/MM/YYYY or DD/MM format, description: To Date of the EVN Bill\>
	* **Success Response:**
	    * **Code:** 200 <br />
	    * Incase of a properly made request, the response would follow schema.

			```
			{
				"status" : "success",
				"statusCode" : "200",
				"result" : <resultObject>
			}
			```

		    The `resultObject` has the following Schema :

			```
			{
				"name": true/false,
				"address": true/false,
				"evn-id": true,
				"from-date": true/false,
				"to-date": true/false,
				"amount": true
			}
			```

	* **Error Response:**
		There are 3 types of request errors and `HTTP Status Code 400` is returned in all 3 cases:
		- No Image input

			```
			{
				"status": "failure",
				"statusCode": "400",
				"error": "API call requires atlest one input image"
			}
			```

		- More than 1 image input

			```
			{
				"status": "failure",
				"statusCode": "400",
				"error": "API call handles only upto 1 image"
			}
			```

		- Larger than allowed image input

			```
			{
				"status": "failure",
				"statusCode": "400",
				"error": "image size cannot be greater than 6MB"
			}
			```

		All error messages follow the same syntax with the statusCode and status also being a part of the response body, and `string` error message with the description of the error.

		**Server Errors**
		We try our best to avoid these errors, but if by chance they do occur the response code will be 5xx.

	* **Sample Calls:**

		 - verifyEVN

			    curl -X POST https://apac.docs.hyperverge.co/v1.1/verifyEVN \
					  -H 'appid: xxx' \
					  -H 'appkey: yyyy' \
					  -H 'content-type: multipart/form-data;' \
					  -F 'image1=@image_or_pdf_path.png'\
					  -F 'name=Name' \
					  -F 'address=Address' \
					  -F 'evnId=EVN Id' \
					  -F 'fromDate=15/10/2015' \
					  -F 'toDate=15/11/2015' \
					  -F 'amount=100'

## Supported Document Types
|Types|Fields|
---|---
|id_front| id, name, dob, province
|id_back| doi, province
|id\_new\_front| id, name, gender, dob, address, doi, doe, province
|id\_new\_back| doi, doe
|mrc_front| mrc-number
|mrc_back| name, address, brand, model, capacity, engine-number, chassis-number, number-plate, price, verification: (name, month-current-registration, year-current-registration, month-first-registration, year-first-registration)
|dl\_old\_front| dl-number, name, dob, nationality, address, expiry
|dl\_new\_front| dl-number, name, dob, nationality, address, expiry

### Response Structure for Each Type:
- #### National ID
	- type: **id_front**

	   	```
	   	{
	    	"id": <type: String, description: ID of the holder>,
	    	"name": <type: String, description: Name of the holder>,
	    	"dob": <type: String, description: Date of Birth of the holder>,
	    	"province": <type: String, description: Province of Issue of the ID>,
		"fraudCheck": 
		  {
		    "isBlackWhite": <type:String "yes/no", description: yes if card is black and white>,
		    "provinceMismatch": <type:String "yes/no", description: yes if mismatch between ID and province>
		  }
	   	}
	  	```

	- type: **id_back**

	   	```
	   	{
	    	"doi": <type: String, description: Date of Issue of the National ID>,
	    	"province": <type: String, description: Province of Issue of the ID>
	   	}
	   	```

	- type: **id\_new\_front**

	   	```
	   	{
	    	"id": <type: String, description: ID of the holder>,
	    	"name": <type: String, description: Name of the holder>,
	    	"gender": <type: "M/F", description: Gender of the holder>,
	    	"dob": <type: String, description: Date of Birth of the holder>,
	    	"address": <type: String, description: Current Address of the holder>,
	    	"doi": <type: String, description: Date of Issue of the ID>,
	    	"doe": <type: String, description: Date of Expiry of the ID>,
	    	"province": <type: String, description: Province of Issue of the ID>,
		"fraudCheck": 
		  {
		    "isBlackWhite": <type:String "yes/no", description: yes if card is black and white>,
		    "provinceMismatch": <type:String "yes/no", description: yes if mismatch between ID and province>
		  }
	   	}
	  	```

	- type: **id\_new\_back**

		```
		{
			"doi": <type: String, description: Date of Issue of the National ID>,
	    	"doe": <type: String, description: Date of Expiry of the National ID>
	   	}
	   	```


- #### Motorbike Registration Certificate (MRC)
	- type: **mrc_front**

		```
		{
			"mrc-number": <type: String, description: MRC Number of the motorbike>
		}
		```

	- type: **mrc_back**

		```
		{
			"name": <type: String, description: Name of the Owner>,
			"address": <type: String, description: Address of the Owner>,
			"brand": <type: String, description: Brand of the Motorbike>,
			"model": <type: String, description: Model of the Motorbike,
			"capacity": <type: String, description: Capacity of the Motorbike>,
			"engine-number": <type: String, description: Engine Number of the Motorbike>,
			"chassis-number": <type: String, description: Chassis Number of the Motorbike>,
			"number-plate": <type: String, description: Number Plate of the Motorbike>,
			"price": <type: Float, description: Estimated Original Price of the Motorbbike>,
			"verified" : {
				"name": <type: Boolean, description: true if verified, false if not verified>,
				"month-first-registration": <type: Boolean, description: true if verified, false if not verified>,
				"year-first-registration": <type: Boolean, description: true if verified, false if not verified>,
				"month-current-registration": <type: Boolean, description: true if verified, false if not verified>,
				"year-current-registration": <type: Boolean, description: true if verified, false if not verified>,
			}
		}
		```
- #### DL
	- type: **dl_old_front**

	   	```
	   	{
	    	"dl-number": <type: String, description: ID of the holder>,
	    	"name": <type: String, description: Name of the holder>,
	    	"dob": <type: String, description: Date of Birth of the holder>,
	    	"nationality": <type: String, description: Nationality of the holder>,
	    	"address": <type: String, description: Current Address of the holder>,
	    	"expiry": <type: String, description: Date of Expiry of the ID>
	   	}
	  	```
	- type: **dl_new_front**

	   	```
	   	{
	    	"dl-number": <type: String, description: ID of the holder>,
	    	"name": <type: String, description: Name of the holder>,
	    	"dob": <type: String, description: Date of Birth of the holder>,
	    	"nationality": <type: String, description: Nationality of the holder>,
	    	"address": <type: String, description: Current Address of the holder>,
	    	"expiry": <type: String, description: Date of Expiry of the ID>
	   	}
	  	```

## Confidence Score for Prediction

For any field which is extracted from the document, the confidence score would be reported in the key `"conf"`. The score would be a float value between 0 and 1.  The key `"to-be-reviewed"` takes the values `"yes/no"`, `yes` indicates that the field is flagged for manual review.

## Data logging and clientId

These optional params are used to facilitate better debugging of the system. 

`clientId` is a unique identifier that is assigned to the end customer by the API user. This would need to be passed in the request body. And the parameter, would be the 
same for the different API calls made for the of same customer.

By default, the input images are not stored by HyperVerge systems, however, if the user sets the optional parameter `dataLogging` to string value "yes", then the images will be stored and the requestId can be 
provided to HyperVerge to check the uploaded image incase of an inaccurate extraction.

<!---
## API wrappers and sample code snippets (Beta)
1. [Python](samples/python/)
2. [Node.js](samples/node.js/)
3. [Java](samples/java)
4. [PHP](samples/php)
--->

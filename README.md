# Introduction 
The  [Google Certificate Authority Service](https://cloud.google.com/certificate-authority-service) is a highly available, scalable Google Cloud service that enables you to simplify, automate, and customize the deployment, management, and security of private certificate authorities (CA).

# Prerequsites
##[Authentication](https://cloud.google.com/docs/authentication/production)
A JSON file generated for a Google Service Account will need to be created and placed on the AnyGateway Server. The path of this file into the GOOGLE_APPLICATION_CREDENTIALS environment variable to be used during a CA session. Since the 
AnyGateway is requried to run as the Network Service account, the registry will need to be modified to provide the service acess to the Envrionment variable above. The GOOGLE_APPLICATION_CREDENTIALS variable should be placed in the following 
registry location and read access provided:

* HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment

##[Authorization](https://cloud.google.com/certificate-authority-service/docs/reference/permissions-and-roles)
Currently the only method supported for authentication is a Service Credential with the following IAM authorizations. Any built in role with the below authorizations will function as well. 
* privateca.certificateAuthorities.list
* privateca.certificateAuthorities.get
* privateca.certificates.create
* privateca.certificates.get
* privateca.certificates.list
* privateca.certificates.update
* privateca.reusableConfigs.get
* privateca.reusableConfigs.list
* resourcemanager.projects.get

# Configuration
The following sections will breakdown the required configurations for the AnyGatewayConfig.json file that will be imported to configure the Google CA. 

## Templates
The Google CA does not have the concept of Templates; however, the section is still required to map a local template to enrollment parameters in the Google CA. Currently the only
 required parameter is the lifetime of the certificate in days. 
 ```json
   "Templates": {
    "GoogleCAWebServer": {
      "ProductID": "GatewayProductID",/*Required by AnyGateway*/
      "Parameters": {
        "Lifetime": "300",/*days*/
      }
    }
}
 ```
## Security
The security section does not change specificly for the Google CA.  Refer to the [AnyGateway Documentation](https://kfeaus00web-01.corp.keyfactor.com/keyfactordocs/AnyGateway/v20.9/Generic/Content/AnyGateway/Introduction.htm) for more detail
```json
  /*Grant permissions on the CA to users or groups in the local domain.
	READ: Enumerate and read contents of certificates.
	ENROLL: Request certificates from the CA.
	OFFICER: Perform certificate functions such as issuance and revocation. This is equivalent to "Issue and Manage" permission on the Microsoft CA.
	ADMINISTRATOR: Configure/reconfigure the gateway.
	Valid permission settings are "Allow", "None", and "Deny".*/
    "Security": {
        "Keyfactor\\Administrator": {
            "READ": "Allow",
            "ENROLL": "Allow",
            "OFFICER": "Allow",
            "ADMINISTRATOR": "Allow"
        },
        "Keyfactor\\gateway_test": {
            "READ": "Allow",
            "ENROLL": "Allow",
            "OFFICER": "Allow",
            "ADMINISTRATOR": "Allow"
        },		
        "Keyfactor\\SVC_TimerService": {
            "READ": "Allow",
            "ENROLL": "Allow",
            "OFFICER": "Allow",
            "ADMINISTRATOR": "None"
        },
        "Keyfactor\\SVC_AppPool": {
            "READ": "Allow",
            "ENROLL": "Allow",
            "OFFICER": "Allow",
            "ADMINISTRATOR": "Allow"
        }
    }
```
## CerificateManagers
The Certificate Managers section is optional.
	If configured, all users or groups granted OFFICER permissions under the Security section
	must be configured for at least one Template and one Requester. 
	Uses "<All>" to specify all templates. Uses "Everyone" to specify all requesters.
	Valid permission values are "Allow" and "Deny".
```json
  "CertificateManagers":{
		"DOMAIN\\Username":{
			"Templates":{
				"MyTemplateShortName":{
					"Requesters":{
						"Everyone":"Allow",
						"DOMAIN\\Groupname":"Deny"
					}
				},
				"<All>":{
					"Requesters":{
						"Everyone":"Allow"
					}
				}
			}
		}
	}
```
## CAConnection
The CA Connection section will determine which CA in the Google cloud is integrated with Keyfactor. There are 3 required configuration fields
* ProjectId  
This is the Resource ID of the project that contains the Google CA Service
* LocationId  
This is the resource ID of the geograpical location (i.e. us-east1) within the Google Cloud
* CAId  
This is the resource Id of the CA created using the [Google Cloud Console](https://console.cloud.google.com)

```json
"CAConnection": {
    "ProjectId": "concise-frame-296019",
    "LocationId": "us-east1",
    "CAId":"ca-enterprise-subordinate-sandbox-tls",
}
```
## GatewayRegistration
There are no Google Specific Changes for the GatewayRegistration section. Refer to the [AnyGateway Documentation](https://kfeaus00web-01.corp.keyfactor.com/keyfactordocs/AnyGateway/v20.9/Generic/Content/AnyGateway/Introduction.htm) for more detail
```json
  "GatewayRegistration": {
    "LogicalName": "GoogleCASandbox",
    "GatewayCertificate": {
      "StoreName": "CA",
      "StoreLocation": "LocalMachine",
      "Thumbprint": "bc6d6b168ce5c08a690c15e03be596bbaa095ebf"
    }
  }
```

## ServiceSettings
There are no Google Specific Changes for the GatewayRegistration section. Refer to the [AnyGateway Documentation](https://kfeaus00web-01.corp.keyfactor.com/keyfactordocs/AnyGateway/v20.9/Generic/Content/AnyGateway/Introduction.htm) for more detail
```json
  "ServiceSettings": {
    "ViewIdleMinutes": 8,
    "FullScanPeriodHours": 1,
	"PartialScanPeriodMinutes": 60
  }
```

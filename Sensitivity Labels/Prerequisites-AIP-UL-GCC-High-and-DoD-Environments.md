---
# required metadata

title: Prerequisites for AIP UL GCC-High and DoD Environments
description: Prerequisites for AIP UL GCC-High and DoD Environments

---

# Prerequisites for AIP UL GCC-High and DoD Environments 
>*[Source Article ](https://github.com/MicrosoftDocs/EMDocs/blob/master/EMDocs/Solutions/ems-aip-premium-govt-service-description.md).*

## Parity with Azure Information Protection Premium Commercial Offerings 

Known existing gaps between Azure Information Protection Premium GCC High/DoD and the commercial offering, as of July 2020 are:

* Document tracking and revocation are currently not available. 

* The classification and labeling add-in is only supported for Microsoft 365 Apps (version 9126.1001 or higher). Office 2010, Office 2013, and other Office 2016 versions are not supported. 

* Information Rights Management (IRM) is supported only for Microsoft 365 Apps (version 9126.1001 or higher). Office 2010, Office 2013, and other Office 2016 versions are not supported. 

* Sharing of protected documents and emails to users in the commercial cloud is not currently available. Includes Microsoft 365 Apps users in the commercial cloud, non-Microsoft 365 Apps users in the commercial cloud, and users with an RMS for Individuals license. 

* Information Rights Management with SharePoint Online (IRM-protected sites and libraries) is currently not available. 

* The Rights Management Connector is currently not available.

* The Mobile Device Extension for AD RMS is currently not available.

## Configuring Azure Information Protection unified labeling client for GCC High and DoD customers

> [!IMPORTANT]
> As of the July 2020 update, all *new* GCC High customers of the Azure Information Protection unified labeling solution, can make use of both General menu and Scanner menu features only. 

### AIP apps configuration (unified labeling )

For the unified labeling solution, AIP apps on Windows need a special registry key to point them to the correct sovereign cloud. Make sure to use the correct values for your setup.  

| Registry Node | HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\MSIP |
| --- | --- |
| Name | CloudEnvType |
| Value | 0/1/2 |
| Type | REG_DWORD |
 

| Values  | details  |
|---------|---------|
|GCC   |     1    |
|GCC High    |    2     |


 
- Default value of the registry key is 0 (zero). 
- If the key is empty or incorrect, the expected behavior is a print error into the log, and it will behave like the default (0-commercial). 
- If the key doesn’t exist, it will behave as default (0-commercial).
- Make sure not to delete the registry key after an uninstall. 

## Configuring Azure Information Protection for GCC High and DoD customers

The following configuration details are relevant for all Azure Information Protection solutions for GCC High and DoD customers, including unified labeling solutions. 




### Enable Rights Management for the tenant

For the encryption to work correctly, the Rights Management Service must be enabled for the tenant.

* Check if the Rights Management service is enabled
  * Launch PowerShell as an Administrator
  * Run `Install-Module aadrm` if the AADRM module is not installed 
  * Connect to service using `Connect-aadrmservice -environmentname azureusgovernment`
  * Run `(Get-AadrmConfiguration).FunctionalState` and check if the state is  `Enabled`
* If the functional state is `Disabled`, run `Enable-Aadrm`

### DNS configuration for encryption (Windows)
For encryption to work correctly, Office client applications must connect to the GCC, GCC High/DoD instance of the service and bootstrap from there. To redirect client applications to the right service instance, the tenant admin must configure a DNS SRV record with information about the Azure RMS URL. Without the DNS SRV record, the client application will attempt connect to the public cloud instance by default, and fail.

Also, the assumption is that users will log in with the username based off the tenant-owned-domain (for example: joe@contoso.us), and not the onmicrosoft username (for example: joe@contoso.onmicrosoft.us). The domain name from the username is used for DNS redirection to the right service instance.

* Get the Rights Management Service ID 
  * Launch PowerShell as an Administrator 
  * If the AADRM module is not installed, run `Install-Module aadrm`  
  * Connect to service using `Connect-aadrmservice -environmentname azureusgovernment`
  * Run `(Get-aadrmconfiguration).RightsManagementServiceId` to get the Rights Management Service ID
* Sign in to your DNS provider, and navigate to the DNS settings for the domain to add a new SRV record
  * Service = `_rmsredir` 
  * Protocol = `_http` 
  * Name = `_tcp` 
  * Target = `[GUID].rms.aadrm.us`  (where GUID is the Rights Management Service ID) 
  * Port = `80` 
  * Priority, Weight, Seconds, TTL = default values 
* Associate the custom domain with the tenant in the [Azure portal](https://portal.azure.us/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Domains). Associating the custom domain will add an entry in DNS, which may take a few minutes to verify after adding the value.   
* Sign in to the Office Admin Center with the corresponding global admin credentials and add the domain (example: contoso.us) for user creation. In the verification process, some more DNS changes might be required. Once verification is done, users can be created.

### DNS configuration for encryption (Mac, iOS, Android)
* Sign in to your DNS provider, and navigate to the DNS settings for the domain to add a new SRV record
  * Service = `_rmsdisco` 
  * Protocol = `_http` 
  * Name = `_tcp` 
  * Target = `api.aadrm.us` 
  * Port = `80` 
  * Priority, Weight, Seconds, TTL = default values 

### AIP apps configuration

The AIP apps on Windows need a special registry key to point them to the right service instance for GCC High/DoD.  

| Registry Node | HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\MSIP |
| --- | --- |
| Name | WebServiceUrl |
| Value | https://api.informationprotection.azure.us |
| Type | REG_SZ (String) |


## Firewalls and network infrastructure

If you have a firewall or similar intervening network devices that are configured to allow specific connections, use these settings to ensure smooth communication for Azure Information Protection.

- To enable the Azure Information Protection Classic client to download labels and label policies: Allow the URL **api.informationprotection.azure.us** over HTTPS.

- Do not terminate the TLS client-to-service connection to the **rms.aadrm.us** URL (for example, to perform packet-level inspection). 

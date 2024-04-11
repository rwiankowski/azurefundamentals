# Part 3 - Identities

Topics:
- Azure and Entra ID
- What is Entra ID
- Entra ID Security Principals
- Multi-Domain Setup
- Entra ID RBAC
- Entra ID Security

## Azure and Entra ID

Historically, when we got to the part where we talk about identities in Azure, I would say:

>Azure starts with Azure Active Directory (which we often call AzureAD or use the acronym AAD). The name of the service is, however, misleading. While we cannot use Azure without Azure AD, it is not a part of Azure, and many customers use it without ever thinking of deploying any Azure Resources. Azure Active Directory is also the backbone of other cloud-based services offered by Microsoft, like Microsoft 365 and Dynamics 365. 

And as we went on, I would keep repeating:

>Azure AD is not Azure!

Not too long ago, however, Micosoft rebranded Azure AD to Entra ID. And while I'm usually cynical about rebranding exercises, this one does the service justice. I mention this because such changes take a long time to solidify in the minds of people who have been using this technology for years. With that in mind, you'll probably still hear me and others use the name Azure AD, so please remember:

> Azure AD == Entra ID

From a technical perspective, though, the name change doesn't mean much, and you still need Entra ID to use Azure - there is no other way. Even if you want to use a third-party identity management solution like Okta, you will still need to use Entra ID in between.

And before we move on, just one more reminder:

>Entra ID is not Azure!

## What is Entra ID

Entra ID is a Software-as-a-Service (SaaS) enterprise Identity and Access Management (IAM) solution. It is a cloud-based and multitenant service that offers:
- **Authentication** - it verifies users' identities and issues access tokens,
- **Authorisation** - it verifies users' access permissions to Enterprise Applications, which can be published via AAD.

Entra ID also offers a wide range of advanced security, collaboration, and other features besides the core identity and access capabilities. Notable mentions include:

- Single Sign-On
- Application Management (similar to Okta)
- Device Management (+Intune)

### Entra ID Glossary

Azure Active Directory introduces a fair bit of new terminology, and it is fundamental to understand what is what. We often use synonyms when referring to the same thing, so become familiar with the essential glossary below.

- **Tenant** - An essential word in our dictionary is the tenant. The tenant (often called a directory or domain) is a dedicated instance of Entra ID. Most commonly, it is intended for use by a specific organisation, but organisations can create additional tenants. As you probably remember from the previous chapter, all subscriptions belong to a single tenant - it is a tree-like structure. Therefore, creating several directories can make your governance very complex. One reasonable exception is separating end-user services like Microsoft 365 and infrastructure-focused Azure into different tenants. Due to the architecture of Entra ID (more on this very soon), such a setup can help with the separation of duties, especially when parts of IT are outsourced. A new ADD tenant was created for you when you signed up for the Azure Free account.
- **Identity** - An Identity is an entity that can be authenticated, for example, a user or an application.
- **Account** - An identity that has data associated with it.
- **Entra ID account** - An identity created through Entra ID or another Microsoft cloud service, such as Microsoft 365. Identities are stored in Entra ID and accessible to your organisation's cloud service subscriptions. This account is also sometimes called a Work or school account.

### Domains and Custom Domains

Every Entra ID tenant has a domain name used to identify the instance of the Azure Active Directory. The initial (mandatory) domain name follows the format:

> \<something of your choice\>.onmicrosoft.com

The domain name must be globally unique because ADD is a multitenant SaaS offering.

The built-in domain name will always be there with you, but you can also add a custom domain name to make your Accounts' User Principal Names (UPNs) more user-friendly.

### Entra ID vs. AD

![Entra ID at a Gance](Images/entraAtAGlance.png)

Active Directory, also known as Active Directory Domain Services (AD DS) or Windows Server Active Directory (WS AD), is a service provided as part of the Windows Server operating system. It acts as a data store for users and devices on a local network and provides authentication and authentication mechanisms. While that description could also fit Azure Active Directory (and there is this naming similarity), it is a fundamentally different service. The key differences are shown in the following table:

| Characteristic | Entra ID | WS AD |
| -------------- | -------- | ----- |
| Structure | Flat - no Organisational Units (OUs) or Groups Policy Objects (GPOs) | Tree-like, with OUs and GPOs |
| Queried using | REST API over HTTP | LDAP |
| Authentication protocol(s) | SAML, WS-Fed, OpenID Connect, OAuth | Kerberos |
| Single Sign-On | Native | Requires AD FS |


### Entra ID Editions

In its default form, Entra ID is a free service. However, the free edition has limited benefits and features, so you will want to upgrade to Entra ID Premium in most production scenarios. To understand why, we will look at the following table, which provides an overview of the differences between various Entra ID editions:


| Feature | Entra ID Free - Security defaults (enabled for all users) | Entra ID Free - Global Administrators only | Office 365 | Entra ID Premium P1 | Entra ID Premium P2 |
|---------|----------------------------------------------------------|--------------------------------------------|------------|---------------------|---------------------|
| Protect Entra ID tenant admin accounts with MFA | ● | ● (Entra ID Global Administrator accounts only) | ● | ● | ● |
| Mobile app as a second factor | ● | ● | ● | ● | ● |
| Phone call as a second factor |  | ● | ● | ● | ● |
| SMS as a second factor |  | ● | ● | ● | ● |
| Admin control over verification methods |  | ● | ● | ● | ● |
| Fraud alert |  |  |  | ● | ● |
| MFA Reports |  |  |  | ● | ● |
| Custom greetings for phone calls |  |  |  | ● | ● |
| Custom caller ID for phone calls |  |  |  | ● | ● |
| Trusted IPs |  |  |  | ● | ● |
| Remember MFA for trusted devices	| 	| ●	| ●	| ●	| ● |
| MFA for on-premises applications	| 	|	|	| ●	| ● |
| Conditional access	| 	|	|	| ●	| ● |
| Risk-based conditional access	| 	|	|	|	| ● |
| Identity Protection (Risky sign-ins, risky users)	| 	|	|	|	| ● |
| Access Reviews	| 	|	|	|	| ● |
| Entitlements Management	| 	|	|	|	| ● |
| Privileged Identity Management (PIM), just-in-time access	| 	|	|	|	| ● |
| Lifecycle Workflows (preview)	| 	|	|	|	| ● |

The table, taken from Microsoft's official documentation, could and should be more precise, especially on MFA. In a way, it hides the fact that Multi-Factor Authentication is not included in the free tier. At this point, you might feel the urge to point at the table and correct me. However, the Azure Active Directory free edition only provides MFA for accounts with administrative permissions in the tenant. And that group should be as small as possible.
To get MFA and another handy security feature called Conditional Access, you will need a P1 license. That should be your default for a regular user. However, considering the components required for people who manage Entra ID and Azure Resources, you should go for P2, as it gives you Privileged Access Management.

We will take a closer look at those security features very soon, but first, we need to get more familiar with a few other topics. 

## Entra ID Security Principals

A Security Principal is an Entra ID object (or its logical representation) and can be represented by the following:
- Users - cloud-native created directly in Entra ID, synchronised from Windows Server Active Directory or guest,
- Groups - security or Microsoft 365 enabled,
- Service Principals - also known as SPNs, act as service accounts used by apps,
- Managed Identities - which represent SPNs that are managed by Azure.

## Multi-Domain Setup

Microsoft will (almost) always recommend having only a single Entra ID tenant. They have good reasons to provide such recommendations, and in many cases, it's the best option. But there are situations in which you need to use several AAD domains.

Those special situations include the following:
- As part of a regulatory compliance framework, you need to separate the tenant of your production environment from any non-production environment.
- You work for a globally distributed enterprise, and data residency laws require storing data within a specific geographical region. 
- You plan on outsourcing parts of your IT landscape to a 3rd party and want to retain a clear separation of duties. Remember - Entra ID has a flat structure, and some permissions can only be granted on the scope of the entire directory. AUs help with some things, but often we need more.
- You work for a managed service provider and must use a separate tenant for each customer.

I excluded from the list above a sandbox tenant you might use individually or in a shared setup with a limited group to learn and try new features and services. That is an extra one for the considerations above.

*Important - If you choose or are forced to use a multitenant setup, I strongly recommend keeping the number of directories to the minimum and paying particular attention to securing each one with adequate measures. Each of those tenants is a liability!*

### Entra ID Business-to-Business Collaboration

![Entra ID B2B Collaboration Overview](Images/b2bCollaborationOverview.png)

When you end up having multiple tenants, you could keep them completely isolated and create duplicate user accounts (and potentially groups) in every one of them, but that would be impractical. Also, you would eventually face configuration drift and potentially expose security vulnerabilities.

Thankfully, we have Entra ID Business to Business (Entra ID B2B) collaboration to mitigate this challenge. The service allows us to invite an Entra ID account from one tenant as a guest user in another tenant. If you remember Guest Users from earlier in this part, that was Entra ID B2B.
When you invite an external account into your tenant, your directory only stores a reference (pointer) to the Entra ID account in another tenant. When the user tries to authenticate, they are redirected to their home tenant, and once they obtain an authentication token, they are redirected back and authorised.

As a result, you can use a single Entra ID account to access multiple Entra ID tenants. 

This service also works great when you want to give users from partner organisations access (like vendors and service providers) to a part of your Azure environment. Instead of creating (and managing) Entra ID accounts for them, you can invite them and let them use their existing identities.

In the example above, I talk about one Entra ID tenant inviting users from another ADD as guest users, but there are other options. You can also invite Microsoft, Google, Facebook (though with limitations) and other accounts. However, those scenarios are rare, and in many cases, they are not good practice. If you use someone's work or school account, you can be reasonably confident that that account will be disabled or removed when they leave the organisation. On the other hand, inviting someone using their personal Microsoft account will place that responsibility on you. 

### Entra ID Busines to Customer

Azure also offers a Business-to-customer service (Entra ID B2C), but the name can be misleading. You might expect similar behaviour after learning about B2B, but that is untrue. To use Entra ID B2C, you create a new, particular type of Entra ID tenant. That tenant is used solely to manage end-customer identities and their access to your applications. 

## Entra ID RBAC

In the previous part, we looked at how role-based access control works for Azure, but we never mentioned anything about Entra ID. But, since "Entra ID is not Azure", it has its own RBAC stack.

### How it works

Thankfully, Entra ID RBAC works almost the same as with Azure:

>You create an *assignment* of a *role definition* to a *security principal* at a particular *scope*.

The main difference lies in the scope - while Azure can have an elaborate management hierarchy with several levels eligible for assignments, Entra ID is flat. We assign Entra ID RBAC roles to the entire tenant (and sometimes AUs).

### Role Definitions

Because Entra ID is used for all cloud-based services offered by Microsoft, not just Azure, it has a long list of built-in role definitions. Most of those you will probably never need, but some are worth taking a closer look at:

1. Global Administrator can:
 - Manage access to all administrative features in Azure Active Directory, as well as services that federate to Azure Active Directory
 - Assign administrator roles to others
 - Reset the password for any user and all other administrators

2. User Administrator can:
 - Create and manage all aspects of users and groups
 - Manage support tickets
 - Monitor service health
 - Change passwords for users, Helpdesk administrators, and other User Administrators

3. Billing Administrator can:
 - Make purchases
 - Manage subscriptions
 - Manage support tickets
 - Monitor service health

Apart from those three, I highly recommend that you check out GlobalReader, Groups Administrator, Application Administrator, and Security Administrator in the [official docs from Microsoft](https://learn.microsoft.com/en-us/azure/active-directory/roles/permissions-reference)

*IMPORTANT - Global Administrator is the highest permission level in Entra ID. The role allows you to do anything - the equivalent of root.
 

### Where Entra ID RBAC meets Azure RBAC

The Global Administrator role is another one of the few places Entra ID and Azure come together. If you have this Entra ID role, you can navigate to the Properties section of the Entra ID blade in the Azure Portal and use the option "Access management for Azure resources". Doing this will give you the User Access Administrator RBAC role in Azure RBAC (on the entire hierarchy). With that, you could create an assignment giving yourself the Owner permissions on any part of the Azure landscape.

![Access management for Azure resources](Images/accessManagement.png)

**Therefore, the Global Administrator role gives you unrestricted access to Entra ID and effectively to the entire Azure hierarchy within the tenant.** 


## Entra ID Security
Finally, we will dive into some security features Entra ID offers. We won't cover everything - we will focus on the two I consider fundamental to maintaining a solid baseline security posture and those you might encounter during the exams. Also, remember that Entra ID is developed very dynamically - Microsoft is probably releasing new features as I write these words, so [check the documentation for the latest and greatest](https://learn.microsoft.com/en-us/azure/security/fundamentals/identity-management-overview). 

### Conditional Access

![Entra ID Conditional Access](Images/azureAdConditionalAccess.png)

One of the fundamental security features of Azure Active Directory is Conditional Access. The feature analyses various signals to automate decisions and enforce organisational access policies.

The signals include, but are not limited to, the following:
- user identity and group membership,
- device platform,
- location,
- sign-in risk,
- client application.

Conditional Access policies are like if-then statements allowing or blocking access, enforcing multi-factor authentication, or restricting the user's session. 

Setting up conditional access is one of the first things I recommend configuring in your Entra ID tenant. Start with a simple set to provide a solid security baseline and build on top of that - limiting access to the Azure Management plane and enforcing MFA for all users should be sufficient. Do exclude your break-glass accounts, though!

### Multi-Factor Authentication

Multi-factor authentication (MFA) is another fundamental security feature of Entra ID. It requires users to verify their identity by providing two or more forms of authentication. Those forms include the following:
- something you know, like your account password
- something you are, like your fingerprint or face/eye scan
- something you have, like a security key or your smartphone

The MFA service provided by Azure Active Directory supports the following forms of additional verification:
- Microsoft Authenticator
- Windows Hello for Business
- FIDO2 security key
- OATH hardware token 
- OATH software token
- SMS
- Voice call

Multi-factor authentication is a relatively simple yet potent protection mechanism. A common saying reminds us that "identity is the new perimeter", and enforcing MFA is the best thing we can do to help secure users in your organisation.

[Back to README](../README.md)
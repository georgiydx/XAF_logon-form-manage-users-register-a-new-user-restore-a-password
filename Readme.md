<!-- default badges list -->
![](https://img.shields.io/endpoint?url=https://codecentral.devexpress.com/api/v1/VersionRange/134075799/22.2.2%2B)
[![](https://img.shields.io/badge/Open_in_DevExpress_Support_Center-FF7200?style=flat-square&logo=DevExpress&logoColor=white)](https://supportcenter.devexpress.com/ticket/details/E4037)
[![](https://img.shields.io/badge/📖_How_to_use_DevExpress_Examples-e9f6fc?style=flat-square)](https://docs.devexpress.com/GeneralInformation/403183)
<!-- default badges end -->
<!-- default file list -->

# XAF Blazor UI: How to extend the logon form - register a new user, restore a password

> **Note**:
> An alternative solution exists: 
>
> [How to: Use Google, Facebook and Microsoft accounts in ASP.NET XAF applications (OAuth2 authentication demo)](https://github.com/DevExpress-Examples/xaf-web-forms-use-oauth2-authentication-providers). 
>
> Instead of a custom-tailored implementation, we recommend that you delegate these routine tasks to OAuth2 providers. Microsoft 365 or Google GSuite services enable user and document management that's familiar to anyone who works with business apps. Your XAF application can easily integrate these OAuth2 providers into the logon form. You only need to add some boilerplate code.
    
## Implementation Overview

This example contains a reusable `Security.Extensions` module that enables the following functionality:

 - [Security - add a capability to register a new user from the logon form](https://supportcenter.devexpress.com/ticket/details/s32938/security-how-to-register-a-new-user-from-the-logon-form)
 - [Security.Authentication - add a "Forgot Password" feature](https://supportcenter.devexpress.com/ticket/details/s33481/security-authentication-provide-a-forgot-password-feature)

![image](https://user-images.githubusercontent.com/14300209/128016215-31fc417a-cfb9-4ce4-910a-e1e215c1c63d.png)

This custom module implements several notable :

- Application Model settings (Model.DesignedDiffs.xafml) that place custom Actions next to the logon form input fields. See [How to: Include an Action to a Detail View Layout](https://docs.devexpress.com/eXpressAppFramework/112816/ui-construction/view-items-and-property-editors/include-an-action-to-a-detail-view-layout). 
- Non-persistent data models for parameter screens (LogonActionParameters.cs) 
- A View Controller (ManageUsersOnLogonController.cs) for the logon Detail View. The controller declares custom Actions and their behavior. See the XafApplication.[CreateCustomLogonWindowControllers](https://documentation.devexpress.com/eXpressAppFramework/DevExpressExpressAppXafApplication_CreateCustomLogonWindowControllerstopic.aspx) event in Module.cs to find controller registration code and other service logic. 

---------------------------------

## Extend the Logon Form in Your Project

In order to use this module in your own project, follow the steps below: 

1. Download and include the `Security.Extensions` module project into your XAF solution and rebuild it. See [How to Add an Existing Project](https://learn.microsoft.com/en-us/previous-versions/ff460187(v=vs.140)?redirectedfrom=MSDN) in MSDN.

2. Invoke the Module Designer for your platform-agnostic module and drag and drop the **SecurityExtensionsModule** from the Toolbox.

3. Add the following code into your platform-agnostic module class:

   ```cs
   static YourPlatformAgnosticModuleName() {
       SecurityExtensionsModule.CreateSecuritySystemUser = Updater.CreateUser;
   } 
   ```
   where 'Updater.CreateUser' is your custom method that matches the following definition:
   ```cs
   public delegate IAuthenticationStandardUser CreateSecuritySystemUser(IObjectSpace objectSpace, string userName, string email, string password, bool isAdministrator);
   ```
   
4. Edit **Startup.cs** in the **SolutionName.Blazor.Server** project. Add the following code into the **ConfigureServices()** method to customize the [SecurityStrategyComplex.AnonymousAllowedTypes](https://docs.devexpress.com/eXpressAppFramework/DevExpress.ExpressApp.Security.SecurityStrategy.AnonymousAllowedTypes) property. Note the **RegisterXPOAdapterProviders()** method call that you would only need in applications that use XPO.

   ```cs
   services.AddXafSecurity(options => {
       options.RoleType = typeof(PermissionPolicyRole);
       // ApplicationUser descends from PermissionPolicyUser and supports OAuth authentication. For more information, refer to the following help topic: https://docs.devexpress.com/eXpressAppFramework/402197
       // If your application uses PermissionPolicyUser or a custom user type, set the UserType property as follows:
       options.UserType = typeof(DXApplication1.Module.BusinessObjects.ApplicationUser);

       // ApplicationUserLoginInfo is only necessary for applications that use the ApplicationUser user type.
       // Comment out the following line if using PermissionPolicyUser or a custom user type.
       options.UserLoginInfoType = typeof(DXApplication1.Module.BusinessObjects.ApplicationUserLoginInfo);
       options.Events.OnSecurityStrategyCreated = securityStrategy => {
           ((SecurityStrategy)securityStrategy).RegisterXPOAdapterProviders(); // Only required for XPO
           ((SecurityStrategy)securityStrategy).AnonymousAllowedTypes.Add(typeof(ApplicationUser));
           ((SecurityStrategy)securityStrategy).AnonymousAllowedTypes.Add(typeof(PermissionPolicyRole));
           ((SecurityStrategy)securityStrategy).AnonymousAllowedTypes.Add(typeof(ApplicationUserLoginInfo));
   };
   ```
   
## Files to Review

**NOTE**: Implementation details do not depend on your ORM tool of choice. As such, the following two sets of files contain the same code. We added both lists only for your convenience - so you can navigate to the folder you need without hesitation.

EF Core:

* [Updater.cs](./CS/EFCore/DXApplication1.Module/DatabaseUpdate/Updater.cs)
* [LogonActionParameters.cs](./CS/EFCore/Security.Extensions/LogonActionParameters.cs)
* [ManageUsersOnLogonController.cs](./CS/EFCore/Security.Extensions/ManageUsersOnLogonController.cs) 
* [Module.cs](./CS/EFCore/Security.Extensions/Module.cs)
* [Startup.cs](./CS/EFCore/DXApplication1.Blazor.Server/Startup.cs)

XPO:

* [Updater.cs](./CS/XPO/DXApplication1.Module/DatabaseUpdate/Updater.cs)
* [LogonActionParameters.cs](./CS/XPO/Security.Extensions/LogonActionParameters.cs)
* [ManageUsersOnLogonController.cs](./CS/XPO/Security.Extensions/ManageUsersOnLogonController.cs) 
* [Module.cs](./CS/XPO/Security.Extensions/Module.cs)
* [Startup.cs](./CS/XPO/DXApplication1.Blazor.Server/Startup.cs)

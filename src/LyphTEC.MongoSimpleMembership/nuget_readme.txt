MongoSimpleMembership v1.0.2
----------------------------

SimpleMembership providers (Membership & Role) using MongoDB as the backing store.
Project site: http://lyphtec.github.io/MongoSimpleMembership/

You must complete the following steps before your app will run:

1. Configure your data connection string to the MongoDB database

        eg.

        <connectionStrings>
            <clear/>
            <add name="MongoSimpleMembership" connectionString="mongodb://localhost/SimpleMembership?safe=true"  />
        </connectionStrings>

        Note that the connection string must also include the database name. In the example above, this will use the database named "SimpleMembership".

2. Update the system.web/membership & system.web/roleManager sections to use the connection string name as defined above
        
        Also make sure that the "defaultProvider" attributes are set to use the matching MongoSimpleMembership provider names.

        eg.

        <membership defaultProvider="MongoSimpleMembershipProvider">
            <providers>
                <clear />
                <add name="MongoSimpleMembershipProvider" type="LyphTEC.MongoSimpleMembership.MongoSimpleMembershipProvider, LyphTEC.MongoSimpleMembership" connectionStringName="MongoSimpleMembership" />
            </providers>
        </membership>

        <roleManager enabled="true" defaultProvider="MongoRoleProvider">
            <providers>
                <clear />
                <add name="MongoRoleProvider" type="LyphTEC.MongoSimpleMembership.MongoRoleProvider, LyphTEC.MongoSimpleMembership" connectionStringName="MongoSimpleMembership" />
            </providers>
        </roleManager>

3. If you are using the default "ASP.NET MVC4 Internet" template. You must make some changes to AccountController:
        
        a. Remove the [InitializeSimpleMembership] attribute (the ActionFilterAttribute defined in Filters/InitializeSimpleMembershipAttribute.cs).
           This is used by the default SimpleMembershipProvider that needs to initialize SQL Server.
           Since we are not using SQL Server, this is no longer required.

        b. Make changes to the ExternalLoginConfirmation() method to remove Entity Framework related hooks used by the default SimpleMembershipProvider.
           
           Here's a complete example of what it should look like after the change:

                [HttpPost]
                [AllowAnonymous]
                [ValidateAntiForgeryToken]
                public ActionResult ExternalLoginConfirmation(RegisterExternalLoginModel model, string returnUrl)
                {
                    string provider = null;
                    string providerUserId = null;

                    if (User.Identity.IsAuthenticated || !OAuthWebSecurity.TryDeserializeProviderUserId(model.ExternalLoginData, out provider, out providerUserId))
                    {
                        return RedirectToAction("Manage");
                    }

                    if (ModelState.IsValid)
                    {
                        // check if user exists
                        if (!WebSecurity.UserExists(model.UserName))
                        {
                            // TODO : Add custom user profile logic here

                            // this will create a non-local account
                            OAuthWebSecurity.CreateOrUpdateAccount(provider, providerUserId, model.UserName);
                            OAuthWebSecurity.Login(provider, providerUserId, createPersistentCookie: false);

                            return RedirectToLocal(returnUrl);
                        }
                        else
                        {
                            ModelState.AddModelError("UserName", "User name already exists. Please enter a different user name.");
                        }
                    }

                    ViewBag.ProviderDisplayName = OAuthWebSecurity.GetOAuthClientData(provider).DisplayName;
                    ViewBag.ReturnUrl = returnUrl;
                    return View(model);
                }

        See the sample MVC4 app at https://github.com/lyphtec/MongoSimpleMembership/tree/master/src/LyphTEC.MongoSimpleMembership.Sample for reference
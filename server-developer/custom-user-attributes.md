# Custom User Attributes

You can add custom user attributes to the registration page and account management console with a custom theme. This chapter describes how to add attributes to a custom theme, but you should refer to the [Themes](https://wjw465150.gitbooks.io/keycloak-documentation/content/server\_development/topics/themes.html#\_themes) chapter on how to create a custom theme.

#### Registration Page <a href="#registration_page" id="registration_page"></a>

To be able to enter custom attributes in the registration page copy the template `themes/base/login/register.ftl` to the login type of your custom theme. Then open the copy in an editor.

As an example to add a mobile number to the registration page add the following snippet to the form:

```html
<div class="form-group">
   <div class="${properties.kcLabelWrapperClass!}">
       <label for="user.attributes.mobile" class="${properties.kcLabelClass!}">Mobile number</label>
   </div>

   <div class="${properties.kcInputWrapperClass!}">
       <input type="text" class="${properties.kcInputClass!}"  id="user.attributes.mobile" name="user.attributes.mobile"/>
   </div>
</div>
```

To see the changes make sure your realm is using your custom theme for the login theme and open the registration page.

#### Account Management Console <a href="#account_management_console" id="account_management_console"></a>

To be able to manage custom attributes in the user profile page in the account management console copy the template `themes/base/account/account.ftl` to the account type of your custom theme. Then open the copy in an editor.

As an example to add a mobile number to the account page add the following snippet to the form:

```html
<div class="form-group">
   <div class="col-sm-2 col-md-2">
       <label for="user.attributes.mobile" class="control-label">Mobile number</label>
   </div>

   <div class="col-sm-10 col-md-10">
       <input type="text" class="form-control" id="user.attributes.mobile" name="user.attributes.mobile" value="${(account.attributes.mobile!'')?html}"/>
   </div>
</div>
```

To see the changes make sure your realm is using your custom theme for the account theme and open the user profile page in the account management console.

\

When working with several sandboxes, you need an easy way to keep your sandboxes in sync without having to refresh the sandboxes. You could use change sets, but you'll need to upload *n* change sets, which can get cumbersome and slow. Use this build file to deploy metadata to several sandboxes at once.


##How To Use

###Add your credentails
This line in the build.xml file is a reference to the file holding your credentials:

```
<property file="build.properties"/>
```

This file is ignored in git because it will hold your usernames and passwords, so you'll need to create it for yourself on your machine. At the base of this repository, create a file called **build.properties**. Here's an example:

```
qa_username = me@mycompany.com.qa
qa_password = QApassAndSecurityToken

uat_username = me@mycompany.com.uat
uat_password = uatPassAndSecurityToken

prod_username = me@mycompany.com
prod_password = productionPassAndSecurityToken

//retrieve
username = me@mycompany.com.mySandboxe
password = passAndToken
endpoint = test.salesforce.com
```

Each of these variables correspond to a merge field in the build.xml file. In the above example, when the build runs then **{$qa_username}** will be replaced with **me@mycompany.com.qa**

**NOTE**: As with all version control, if you accidentally commit sensitive information, immediately revert the changes and then change your password!

###Build your package
Prepare the **package.xml** file to pull down the metadata that you want.

In the **build.properties** file add the credentials of the org you want to retrieve from so that they are used the they **retrieve** task.

###Description of Targets

To run any of these target, run *ant target* in the shell. For example:

```
ant deployToAllSandboxes
```

* **deployToAllSandboxes** - Deploys to all sandboxes. Add or remove tasks from this target to adds more sandboxes. Should any deployment fail, the ones after it won't run
* **deployToProduction** - Deploys production. This will automatically run all tests.
* **validateProduction** - Validates against production. This does not deploy, but if the validation passes then you can utilize [Quick Deploy](http://releasenotes.docs.salesforce.com/en-us/spring15/release-notes/rn_quick_deployment_ga.htm)
* **retrieve** - Retrieves the metadata 

##A Note About Managing Profile Permissions
When deploying fields, you'll also want to deploy the user permissions or else you'll need to do it manually in the destination org. However,
sandboxes do not necessarily have the same user permissions so you may run into issues while deploying field permissions.

Take this build for example:

```
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <types>
        <members>Account.My_Custom_Field__c</members>
        <name>CustomField</name>
    </types>
    <types>
        <members>*</members>
        <name>Profile</name>
    </types>
    <version>32.0</version>
</Package>
```

After running **ant retrieve**, in the profiles you'll notice the User Permissions below Field Permissions:

```
<userLicense>Salesforce</userLicense>
<userPermissions>
    <enabled>true</enabled>
    <name>ActivateContract</name>
</userPermissions>
<userPermissions>
    <enabled>true</enabled>
    <name>ApiEnabled</name>
</userPermissions>
```

If while deploying you run into an error like saying something like 'User permission ApiEnabled does not exist', then you'll need to strip the permissions out of the profile metadata.
Don't worry, this doesn't delete the metadata, it just doesn't include them in the deployment. Use this regex to find all the User Permissions and replace it with a blank.

```
	<userPermissions>[\s\S]*</userPermissions>
```

Now you can run any of your deployments without having to worry about the UserPermissions.


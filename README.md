# Permissions

As Dimensions Online evolved, the need arose to control access to the parts (modules) of Dimensions Online website. 
Serenity provides a basic permissions structure which follows [Repository Pattern](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design "Click to follow link") which as application grew, had limitations.

# Permissions restructure

To address those limitations, the [Feature 24462 (Permission restructure)](https://d-uk.visualstudio.com/Primary/_workitems/edit/24462 "Click to follow link") was introduced. This feature looked into redevopling permissions into a more structured and granular way, in order to better control different modules of Dimensions Online.


## Technical Details

Keeping the future objective of switching to [Microservices](https://microservices.io/ "Dedicated website") the Permissions are now divided into **modules**, with each module taking care of its permissions. The permission [God Object](https://en.wikipedia.org/wiki/God_object "Anti Pattern") is now removed and a new library of permissions are now introduced, with this, each class is named after the module it represents. This follows [SOLID principles](https://en.wikipedia.org/wiki/SOLID "SOLID") for better code quality.

## Class structure

### PermissionKeys Library

1. The classes within `../Modules/Library/PermissionKeys` library are within namepsace of `DimensionsOnline.PermissionKeys` to make them available throughout the project. Each class is decorated with the following two [Attributes](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/attributes/ "Attributes (C#)") at class declaration:

   1. **NestedPermissionKeys** Attribute. The [NestedPermissionKeys](https://serenity.is/Docs/howto/how_to_register_permissions_in_serene    "Serenity link") attribute ensures that permission keys defined here are also registered and shown in permission dialogue.
   2. **DisplayName** Attribute. This sets the custom text for this Permission class, to show in Permission dialogue.
   
2. The `public const string` declaration along with its assignment, denotes a _PermissionKey_ member for the class. This member is decorated with a [Description attribute](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.descriptionattribute?view=netframework-4.8 "DescriptionAttribute (C#)") which displays text for this PermissionKey in Permissions dialgue. The PermissionKey is then used to specify permissions in relevant module/logic flow, e.g. the following PermissionKey represents Supported Person module's configuration criteria of Contact Method (access to edit/modify Supported Person Contact Method configuration).

```csharp
using Serenity.Extensibility;
using System.ComponentModel;

namespace DimensionsOnline.PermissionKeys
{
 /// <summary>
 /// Supported Person (Client) permission keys.
 /// </summary>
 [NestedPermissionKeys] // This is to register Permission and show in Permissions dialogue.
 [DisplayName("Supported Person")] // Custom text for this Permission class, to show in Permission dialogue.
 public class SupportedPerson 
 {
   #region Supported Person (Client).

     /// <summary>
     /// Config: Communication Method - Access to edit/modify Supported Person Communication Method configuration.
     /// </summary>
     [Description("Config: Communication Method - Access to edit/modify Supported Person Communication Method configuration")]
     public const string CommunicationMethod = "SupportedPerson:CommunicationMethod";
     
     // ... Other PermissionKey members go here ... 

   #endregion
 }
}
```
   
   
3. These are then used to apply permission related restrictions on relevant modules/workflows. Couple of examples as below:
   1. **Page access** To grant a permission to access a page, the `xyzPage` is decorated with `PageAuthorize` attribute, passing in appropriate permission:
   ```
   [PageAuthorize(SupportedPerson.View)]
   public class ClientController : Controller
   { /* code goes here */
   ```

4. The PermissionKey members are wrapped in `#region ` outlining feature for better viewing.

**NOTE:** There is a special class `Z_999_CombinedKeys` in this library, this holds combined permission keys from other keys in DimensionsOnline.PermissionKeys namespace. This odd name is because of TypeScript generation limitations, where this is generated at the very end, so that other keys could be accessible to this class.

#### TypeScript files

The classes in PermissionKeys Library are also translated into TyepScript modules, to use on client side. This is acheived by writing a small console app, that reads into the permission classes from specified library and transforms them into TypeScript modules, saving them in `../Modules/Library/PermissionKeys/TypeScriptFiles` folder. 

The modules are named as `DimensionsOnline.PermissionKeys.<ModuleName>` to represent PermissionKey values for the respective module, e.g. the following is showing PermissionKey members for `Administration` module.

```typescript
/** 
* Permission keys for Administration related module(s).
*/
module DimensionsOnline.PermissionKeys.Administration {

 //#region Administration Keys.

 /** Languages & Translations - Access to edit/modify Languages & Translations. */
 export const LanguageAndTranslation: string = "Administration:LanguageAndTranslation";

 /** Roles: Access to edit/modify Roles. */
 export const RolesEdit: string = "Administration:RolesEdit";

 /** Roles: Access to view the Roles page and records. */
 export const RolesView: string = "Administration:RolesView";

 /** User Management: Assign Roles - Access to assign Roles to a user. */
 export const UserManagementRole: string = "Administration:UserManagementRole";

 /** User Management: Users - Access to edit/modify Users. */
 export const UsersEdit: string = "Administration:UsersEdit";

 /** User Management: View - Access to view the User Management page and records. */
 export const UserManagementView: string = "Administration:UserManagementView";

 //#endregion
}

```

These keys are then used in different

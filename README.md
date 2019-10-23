# Permissions

As Dimensions Online evolved, the need arose to control access to the parts (modules) of Dimensions Online website. 
Serenity provides a basic permissions structure which follows [Repository Pattern](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design "Click to follow link") which as application grew, had limitations.

# Permissions restructure

To address those limitations, the [Feature 24462 (Permission restructure)](https://d-uk.visualstudio.com/Primary/_workitems/edit/24462 "Click to follow link") was introduced. This feature looked into redeveloping permissions into a more structured and granular way, in order to better control different modules of Dimensions Online.


## Technical Details

Keeping the future objective of switching to [Microservices](https://microservices.io/ "Dedicated website") the Permissions are now divided into **modules**, with each module taking care of its permissions. The permission [God Object](https://en.wikipedia.org/wiki/God_object "Anti Pattern") is now removed and a new library of permissions are now introduced, with this, each class is named after the module it represents. This follows [SOLID principles](https://en.wikipedia.org/wiki/SOLID "SOLID") for better code quality.

## Class structure

### PermissionKeys Library

1. The classes within `../Modules/Library/PermissionKeys` library are within namepsace of `DimensionsOnline.PermissionKeys` to make them available throughout the project. Each class is decorated with the following two [Attributes](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/attributes/ "Attributes (C#)") at class declaration:

   1. **NestedPermissionKeys** Attribute. The [NestedPermissionKeys](https://serenity.is/Docs/howto/how_to_register_permissions_in_serene    "Serenity link") attribute ensures that permission keys defined here are also registered and shown in permission dialogue.
   2. **DisplayName** Attribute. This sets the custom text for this Permission class, to show in Permission dialogue.
   
2. The `public const string` declaration along with its assignment, denotes a _PermissionKey_ member for the class. This member is decorated with a [Description attribute](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.descriptionattribute?view=netframework-4.8 "DescriptionAttribute (C#)") which displays text for this PermissionKey in Permissions dialogue. The PermissionKey is then used to specify permissions in relevant module/logic flow, e.g. the following PermissionKey represents Supported Person module's configuration criteria of Contact Method (access to edit/modify Supported Person Contact Method configuration).

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
   
   ```csharp
   [PageAuthorize(SupportedPerson.View)] // Only SupportedPerson.View permission is allowed to access this.
   public class ClientController : Controller
   { /* code goes here */
   ```
   2. **Web API access** To grant a permission to an Web API Endpoint, the endpoint within xyzController class is decorated with `ServiceAuthorize` attribute, passing in appropriate permission, for example in ClientController:
   
   ```csharp
   [HttpPost, ServiceAuthorize(SupportedPerson.Create)] // Only SupportedPerson.Create permission is allowed to access this.
   public SaveResponse Create(IUnitOfWork uow, SaveRequest<MyRow> request)
   {
      // Add current user as creator.
      int.TryParse(Authorization.UserId, out int userId);
      return new MyRepository().Create(uow, request, userId);
   }
   ```


4. The PermissionKey members are wrapped in `#region ` outlining feature for better viewing.

**NOTE:** There is a special class `Z_999_CombinedKeys` in this library, this holds combined permission keys from other keys in DimensionsOnline.PermissionKeys namespace. This odd name is because of TypeScript generation limitations, where this is generated at the very end, so that other keys could be accessible to this class.


#### TypeScript files

The classes in PermissionKeys Library are also translated into TyepScript modules, to use on client side. This is achieved by writing a small console app, that reads into the permission classes from specified library and transforms them into TypeScript modules, saving them in `../Modules/Library/PermissionKeys/TypeScriptFiles` folder. 

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

These keys are then used in different client side scenario. Couple of examples as below:

1. **Dialogues/Forms** To setup permission(s) in Serenity's `xyzDialog.ts` file, we call utility methods from GeneralLibrary utility class:

```typescript
private checkPermissions(): void {
   // Set up Dialogue permissions.
   this.generalLibrary.setupButtonPermission(CommonButtonType.DeleteButton, this.toolbar, PermissionKeys.SupportedPerson.Delete);
   if (this.isNew()) {
       this.generalLibrary.setupButtonPermission(CommonButtonType.UpdateButton, this.toolbar, PermissionKeys.SupportedPerson.Create);
   } else {
       this.generalLibrary.setupButtonPermission(CommonButtonType.UpdateButton, this.toolbar, PermissionKeys.SupportedPerson.Edit);
   }
   // Set up Tab permissions.
   this.generalLibrary.setupCommonDialoguePermission(PermissionKeys.SupportedPerson.Audit, null, this.tabs, "Timeline");
   // Here two tabs are enabled/disabled based upon PermissionKeys.SupportedPerson.Edit access.
   this.generalLibrary.setupCommonDialoguePermission(PermissionKeys.SupportedPerson.Edit, null, this.tabs, "KeySupport", "Personal");
}        
```

2. **Grids/Lists** To setup permission(s) in Serenity's `xyzGrid.ts` file, we call utility methods from GeneralLibrary utility class within Serenity's `createToolbarExtensions():void` method (or any other method where this.toolbar is an object and not `undefined` value) as below:

```typescript
protected createToolbarExtensions(): void {
   super.createToolbarExtensions();
   // Set up common toolbar buttons for grid, here only PermissionKeys.SupportedPerson.Create permission has access to "new" button.
   this.generalLibrary.setupCommonButtonsPermission(SerenityControlType.Grid, this.toolbar, PermissionKeys.SupportedPerson.Create);
}
```

3. **Anchors** The [anchors](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a "Anchor element") or "links" or "hyperlinks" are enabled/disabled differently, because HTML `disabled` attribute doesn't work on them. We have an TypeScript [interface](https://www.typescriptlang.org/docs/handbook/interfaces.html "Interface") for denoting this behaviour:

```typescript
/** Link behaviour in Serenity. */
export interface ILinkBehaviour {
  /** CSS class for the link. */
  cssClass?: string;
  /** Attribute to set if link is to be made disabled. */
  disabledAttribute?: string;
  /** HTML href attribute string for this link. */
  href?: string;
}
```
And we have a utility method, returning an object with information to make an anchor show as 'disabled' on page:

```typescript
/** Returns ILinkBehaviour to disable hyperlinks in Serenity. */
public getDisabledBehaviourForLink(): ILinkBehaviour {

   // Set values for disabled link.
   let behaviour: ILinkBehaviour = {
       cssClass: "disabled"
   };
   // Ref: https://stackoverflow.com/a/48447130/1659999
   behaviour.disabledAttribute = `disabled onclick="(function (e) { e.preventDefault(); })(event)"`;
   // Change behaviour. Ref: https://stackoverflow.com/a/55060009/1659999
   behaviour.href = "javascript:void(0)";

   return behaviour;
}
```

We then use it in a `xyzFormatter.ts` Serenity class (here it is `PostStaffTransferActionsFormatter` class) as below:

```typescript

namespace DimensionsOnline.Organisation {

@Serenity.Decorators.registerFormatter()
export class PostStaffTransferActionsFormatter
    implements Slick.Formatter, Serenity.IInitializeColumn {

    private generalLibrary = new GeneralLibrary();
    private approveLink: ILinkBehaviour;
    private cancelLink: ILinkBehaviour;
    private disabledLink: ILinkBehaviour;

    constructor() {

        if (this.generalLibrary.hasUserPermission(PermissionKeys.OrganisationalUnit.AllColleagueTransferAction)) {

            // Format approval and cancel links.
            let href: string = "#";
            let disabled: string = "";
            this.approveLink = {
                cssClass: "text-underline transfer-approve",
                disabledAttribute: disabled,
                href: href
            };

            this.cancelLink = {
                cssClass: "text-underline transfer-cancel",
                disabledAttribute: disabled,
                href: href
            };
        } else {
            // Not allowed to action, disable links.
            this.disabledLink = this.generalLibrary.getDisabledBehaviourForLink();
            this.approveLink = this.cancelLink = this.disabledLink;
        }
    }

    format(_: Slick.FormatterContext): string {

        const space: string = "&nbsp;&nbsp;&nbsp;";
        let status: string = "";
        status += `<a href="${this.approveLink.href}" class="${this.approveLink.cssClass}" ${this.approveLink.disabledAttribute}>Approve Transfer</a>${space}`;
        status += `<a href="${this.cancelLink.href}" class="${this.cancelLink.cssClass}" ${this.cancelLink.disabledAttribute}>Cancel Transfer</a>${space}`;

        return status;
    }

    initializeColumn(_: Slick.Column): void {
    }
}
}
```

# Permissions

As Dimensions Online evolved, the need arose to control access to the parts (modules) of Dimensions Online website. 
Serenity provides a basic permissions structure which follows [Repository Pattern](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design "Click to follow link") which as application grew, had limitations.

# Permissions restructure

To address those limitations, the [Feature 24462 (Permission restructure)](https://d-uk.visualstudio.com/Primary/_workitems/edit/24462 "Click to follow link") was introduced. This feature looked into redevopling permissions into a more structured and granular way, in order to better control different modules of Dimensions Online.


## Technical Details

Keeping the future objective of switching to [Microservices](https://microservices.io/ "Dedicated website") the Permissions are now divided into **modules**, with each module taking care of its permissions. The permission [God Object](https://en.wikipedia.org/wiki/God_object "Anti Pattern") is now removed and a new library of permissions are now introduced, with this, each class is named after the module it represents. This follows [SOLID principles](https://en.wikipedia.org/wiki/SOLID "SOLID") for better code quality.

### Class structure

1. The classes within `Modules/Library/PermissionKeys` library are within namepsace of `DimensionsOnline.PermissionKeys` to make them available throughout the project. Each class is decorated with the following two [Attributes](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/attributes/ "Attributes (C#)") at class declaration:

   1. **NestedPermissionKeys** Attribute. The [NestedPermissionKeys](https://serenity.is/Docs/howto/how_to_register_permissions_in_serene    "Serenity link") attribute ensures that permission keys defined here are also registered and shown in permission dialogue.
   2. **DisplayName** Attribute. This sets the custom text for this Permission class, to show in Permission dialogue.
   
2. The `public const string` declaration along with its assignment, denotes a _PermissionKey_ member for the class. This member is decorated with a [Description attribute]() which displays text for this PermissionKey in Permissions dialgue. The PermissionKey is then used to specify permissions in relevant module/logic flow, e.g. the following PermissionKey represents Supported Person module's configuration criteria of Contact Method (access to edit/modify Supported Person Contact Method configuration).

```csharp

        /// <summary>
        /// Config: Communication Method - Access to edit/modify Supported Person Communication Method configuration.
        /// </summary>
        [Description("Config: Communication Method - Access to edit/modify Supported Person Communication Method configuration")]
        public const string CommunicationMethod = "SupportedPerson:CommunicationMethod";
```



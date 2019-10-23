# Permissions

As Dimensions Online evolved, the need arose to control access to the parts (modules) of Dimensions Online website. 
Serenity provides a basic permissions structure which follows [Repository Pattern](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design "Click to follow link") which as application grew, had limitations.

# Permissions restructure

To address those limitations, the [Feature 24462 (Permission restructure)](https://d-uk.visualstudio.com/Primary/_workitems/edit/24462 "Click to follow link") was introduced. This feature looked into redevopling permissions into a more structured and granular way, in order to better control different modules of Dimensions Online.


## Technical Details

Keeping the future objective of switching to [Microservices](https://microservices.io/ "Dedicated website") the Permissions are now divided into **modules**, with each module taking care of its permissions. The permission [God Object](https://en.wikipedia.org/wiki/God_object "Anti-Pattern") is now removed and a new library of permissions are now introduced, with each class named after the module it represents.

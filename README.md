Introduction
============
A sample OAUTH resource server. The goal of this project is to provide an API with simple CRUD access for a multi-tenant system.

There are two types of objects that can be managed by this API:
  * Company
  * User

  
Data
====
The data is stored in a mysql table and sample data is included and loaded as part of the launch of the SpringBoot application.

Data flows from the database to the controller in the following manner:

Repository -> Model -> Mapper -> DTO -> Controller


Security
========
Authentication
--------------
Users authenticate to the sample OAUTH AS to obtain a JWT based access token.

Authorization
-------------
The OAUTH AS assigned roles/granted authorities for the user. These are evaluated when accessing the various controllers.
The following table desribes the required roles:

| Controller | Action | ROLE_SUPERUSER | ROLE_COMPANYAMDIN | ROLE_USER |
|:-----------|:-------|      :---:     |       :---:       |   :---:   |
| Company    | Add    |       X        |                   |           |
| Company    | Edit   |       X        |         X         |           |
| Company    | Delete |       X        |                   |           |
| User       | Add    |       X        |         X         |           |
| User       | Edit   |       X        |         X         |     X     |
| User       | Delete |       X        |         X         |           |

A ROLE_SUPERUSER is a static role that grants global rights across all objects. The ROLE_COMPANYAMDIN and ROLE_USER rights 
are assigned to particular objects. For example:
  * **ROLE_COMPANYAMDIN:INITECH** - This grants the client access to perform any action that requires the ROLE_COMPANYAMDIN role but only against the INITECH company.
  * **ROLE_USER:user1@INITECH-USER1** - This grants the client access to perform any action that requires the ROLE_USER role but only against USER1 of the INITECH company.

This is currently implemented in the controller using the @PreAuthorize annotation. It directs the framework to call the RoleChecker.hasValidRole() method passing in the 
value of the company and user that are being passed on the query string.

Field level security
--------------------
The granted roles extend beyond the actions that can be performed against the controller. Specific fields
in the object can only be modified based on the role of the user. The following tables describes the necessary roles:

| Object  | Field        | ROLE_SUPERUSER (read) | ROLE_COMPANYAMDIN (read) | ROLE_USER (read) | ROLE_SUPERUSER (write) | ROLE_COMPANYAMDIN (write) | ROLE_USER (write) |
|:--------|:-------------|:--------------:       |:-----------------:       |:---------:       |:--------------:        |:-----------------:        |:---------:        |
| Company | name         |        X              |         X                |                  |       X                |                           |                   |
| Company | contactName  |        X              |         X                |                  |       X                |         X                 |                   |
| Company | contactEmail |        X              |         X                |                  |       X                |         X                 |                   |
| Company | maxAccounts  |        X              |                          |                  |       X                |         X                 |                   |
| Company | maxsize      |        X              |                          |                  |       X                |                           |                   |
| User    | companyName  |        X              |         X                |                  |       X                |                           |                   |
| User    | login        |        X              |         X                |     X            |       X                |                           |                   |
| User    | password     |        X              |         X                |     X            |       X                |         X                 |     X             |
| User    | quota        |        X              |         X                |                  |       X                |                           |     X             |
| User    | enabled      |        X              |         X                |                  |       X                |         X                 |     X             |

This is is currently implemented in the mapper classes that ModelDTOMapper class. It uses reflections to copy data from the entity to the DTO or vise-versa and analyzes the 
ModelMapper annotations on the properties of the DTO class. For example:

    public class UserDTO {
    	protected String companyName;
        protected String login;
        
        @ModelMapper(readRoles = {"ROLE_SUPERADMIN", "ROLE_COMPANYADMIN"}, writeRoles = {"ROLE_SUPERADMIN", "ROLE_COMPANYADMIN"} )
        protected String password;
    
        @ModelMapper(readRoles = {"ROLE_SUPERADMIN"}, writeRoles = {"ROLE_SUPERADMIN"} )
        protected Long quota;
    
        @ModelMapper(readRoles = {"ROLE_SUPERADMIN"}, writeRoles = {"ROLE_SUPERADMIN"} )
        protected Boolean enabled;
    }

To Do
=====
The primary items that are required are:
# Evalution of the field level security. Is there a better Springy way to implement this?
# Integration tests without the need for the AS
# End to end tests that require the AS (?)
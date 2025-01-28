---
sidebar_label: Overview
slug: /en/cloud/security/cloud-access-management/overview
title: Cloud access management
---
# Access Control in ClickHouse Cloud

ClickHouse Cloud enables customers to manage user access within the console and within the database using either pre-defined roles or custom database roles. 

## Console users and roles
Configure the following role assignments within the Console > Users and roles page.

| Role                   | Description                                      |
|:-----------------------|:-------------------------------------------------|
| Admin                 | Perform all administrative activities for an organization and control all settings. Assigned to the first user in the organization by default. |
| Developer             | View access to everything except Services, ability to generate read-only API keys. |
| Billing               | View usage and invoices, and manage payment methods. |
| Member                | Sign-in only with the ability to manage personal profile settings. Assigned to SAML SSO users by default. |
  
**Service roles** allow users to interact with deployed services and are assigned in addition to console roles. Users may be assigned no services, specific services, or all services. Organization Admins are assigned Service Admin for all services in the organization.

| Role                  | Description                                     |
|:----------------------|:------------------------------------------------|
| Service Admin         | Manage service settings.                        |
| Service ReadOnly      | View services and settings.                     |

## Console service permissions
Configure the following role assignments within the Console > Service > Settings page.

**SQL console roles** allow a console user to access databases within a service using the SQL console. SQL console roles may be assigned to the Service Admin or Service Read Only Console roles. Only users assigned the service role for the specific service will be able to interact with the database via SQL console. 

| Role                  | Description                                     |
|:----------------------|:------------------------------------------------|
| SQL console admin     | Administrative access to databases within the service equivalent to the Default database role. |
| SQL console read only | Read only access to databases within the service |
| Custom                | Configure using SQL [GRANT](/en/sql-reference/statements/grant) statement; assign the role to a SQL console user by naming the role after the user |

To create a custom role for a SQL console user and grant it a general role, run the following commands. The email address must match the user's email address in the console. 
    
1. Create the database_developer role and grant SHOW, CREATE, ALTER, and DELETE permissions.
    
    ```
    CREATE ROLE OR REPLACE database_developer;
    GRANT SHOW ON * TO database_developer;
    GRANT CREATE ON * TO database_developer;
    GRANT ALTER ON * TO database_developer;
    GRANT DELETE ON * TO database_developer;
    ```
    
2. Create a role for the SQL console user my.user@domain.com and assign it the database_developer role.
    
    ```
    CREATE ROLE OR REPLACE `sql-console-role:my.user@domain.com`;
    GRANT database_developer TO `sql-console-role:my.user@domain.com`;
    ```
  
## Database permissions
Configure the following within the services and databases using the SQL [GRANT](/en/sql-reference/statements/grant) statement.

| Role                  | Description                                                                   |
|:----------------------|:------------------------------------------------------------------------------|
| Default               | Full administrative access to services                                        |
| Custom                | Configure using the SQL [GRANT](/en/sql-reference/statements/grant) statement |


Database roles are additive. This means if a user is a member of two roles, the user has the most access granted to the two roles. They do not lose access by adding roles.

Database roles can be granted to other roles, resulting in a hierarchical structure. Roles inherit all permissions of the roles for which it is a member.

Database roles are unique per service and may be applied across multiple databases within the same service.

The illustration below shows the different ways a user could be granted permissions.

![Screenshot 2024-01-18 at 5 14 41 PM](https://github.com/ClickHouse/clickhouse-docs/assets/110556185/94b45f98-48cc-4907-87d8-5eff1ac468e5)

## Database access listings with SQL console users

1. Run the following queries to get a list of all grants in the database. 

    ```
    SELECT grants.user_name,
      grants.role_name,
      users.name AS role_member,
      grants.access_type,
      grants.database,
      grants.table
    FROM system.grants LEFT OUTER JOIN system.role_grants ON grants.role_name = role_grants.granted_role_name
      LEFT OUTER JOIN system.users ON role_grants.user_name = users.name
    
    UNION ALL
    
    SELECT grants.user_name,
      grants.role_name,
      role_grants.role_name AS role_member,
      grants.access_type,
      grants.database,
      grants.table
    FROM system.role_grants LEFT OUTER JOIN system.grants ON role_grants.granted_role_name = grants.role_name
    WHERE role_grants.user_name is null;
    ```
    
2. Associate this list to Console users with access to SQL console.
   
    a. Go to the Console.

    b. Select the relevant service.

    c. Select Settings on the left.

    d. Scroll to the SQL console access section.

    e. Click the link for the number of users with access to the database `There are # users with access to this service.` to see the user listing.

.. _user-guide-user-management-permissions-roles:

Roles Management
================

.. contents:: :local:
    :depth: 3

Overview
--------

Roles are predefined sets of permissions. When you assign a role to a user, you can be sure that the user will be able to access only that information within the system which is necessary for them to do their work. 

.. note:: See a short demo on `how to create a and manage roles <https://www.orocrm.com/media-library/create-manage-roles>`_, or continue reading the step-by-step guidance below.

   .. raw:: html

      <iframe width="560" height="315" src="https://www.youtube.com/embed/jgiKa_rov8Y" frameborder="0" allowfullscreen></iframe>


Roles Creation
^^^^^^^^^^^^^^

Usually roles are created based on the user's job functions: sales manager, marketing team member, administrator. But this is not a strict rule. You can create as many roles as required and configure them according to the needs of your company. 
For how to create a role, see the **Create a Role** section of the the :ref:`Actions with Roles <user-guide-user-management-permissions-roles--actions>` topic.



Role Structure
^^^^^^^^^^^^^^
A role is a set of permissions that you can grant to a user all-in-one. 
There are several nominal types of permissions in OroCRM:

- Permissions to perform a certain action on entity. For each permission of this type you can specify a desired access level.

- Permissions to access system functionalities. They are also called 'capabilities' on the interface. System functionalities belong to the system and thus for them you simply specify whether to include permissions to access them into the role or not.

- Permissions to view workflows and perform transitions.



.. image:: ../img/access_roles_management/user_access.png 


.. _user-guide-user-management-permissions-roles--actions-on-entity-permissions:

Action on Entity Permissions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On each entity in the system a user can perform a certain set of actions. The set may vary for some entities but in general it looks as follows:

+-----------+----------------------------------------------------------------------------+
| Action    | Description                                                                |
+===========+============================================================================+
| View      | A user can see the entity records in the grid and open their view pages.   |
+-----------+----------------------------------------------------------------------------+
| Create    | A user can create a new entity record.                                     |
+-----------+----------------------------------------------------------------------------+
| Edit      | A user can edit entity records.                                            |
+-----------+----------------------------------------------------------------------------+
| Delete    | A user can delete an entity record.                                        |
+-----------+----------------------------------------------------------------------------+
| Assign    | A user can change the owner of an entity record.                           |
+-----------+----------------------------------------------------------------------------+
| Share     | A user can share an entity record with other users.                        |
|           | This action is available only in OroCRM EE.                                |
+-----------+----------------------------------------------------------------------------+
| Configure | A user can set up the system configuration for themselves and other users. |
|           | This action is available on for the **User** entity.                       |
+-----------+----------------------------------------------------------------------------+

For each of this actions you can set an access level, thus defining the range of entity records a user can perform an action on: will these be only the records owned by the user themselves, records of the user's division, all records in the system, etc.?  


The picture below shows the scheme of how permissions for an entity may be configured:

.. image:: ../img/access_roles_management/ex_permissions-on-entity.png 

This is how the corresponding configuration looks on the interface for the **Account** entity:

.. image:: ../img/access_roles_management/roles_permissions_general_ex.png 

For more information about which access levels defines which range, see the :ref:`Access Levels <user-guide-user-management-permissions-roles--acl>` topic.

.. Important::
	Note that the set of available access levels depends on the entity's ownership type. For example, you will not be able to set the **User** access level if the entity's ownership type is **Organization.** Only two access levels are always available: **None**—access is denied and **Global**—access all entity records within the system.
	For more information about ownership types, see the :ref:`Ownership Type <user-guide-user-management-permissions-ownership-type>` section and specifically, the Ownership type and access levels subsection.


Field Level ACLs
""""""""""""""""
All important information that comprises an entity is contained in the entity fields. For example, if you open any record of the **Business Unit** entity, you will see such fields as **Name**, **Organization**, **Description**, **Website**, etc.

When you include the permission to view entity records in a role, users with such role are automatically able to see all fields of the entity.

However, there are situations when it is desirable to hide certain fields from one group of users while still having them available for others. For example, both the sales team and support team require to see **Opportunity** entity records. But as the financial information is often considered sensitive, you may want to hide the **Budget Amount** field from the support team members.


Is is possible to do this using Field Level ACL functionality. When you enable it for an entity, you can assign permissions that allow actions on a particular entity field to a role.

For more information about the field level ACLs, see the `Permissions for an Entity Field (Field Level ACLs) <./access-management-field-level-acl>`__ topic.



System Functionalities Permissions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Permissions of this type either define whether a user must have access to certain parts of the system (e.g., should they see the page with system jobs queue) or extend 'action on entity' permissions (e.g. should a user retain the ability to modify their own user profile if editing of user profiles in general is forbidden for them). Permissions of this type are also called 'capabilities.'' Capabilities can be either enabled or disabled for a role. 

This is how capabilities look on the interface:

.. image:: ../img/access_roles_management/roles_overview2.png

Workflow Permissions
~~~~~~~~~~~~~~~~~~~~

The workflow permissions define whether a user can view a particular workflow and perform transitions within it:

+---------------------+----------------------------------------------------------------------------------------+
| Action              | Description                                                                            |
+=====================+========================================================================================+
| View Workflow       | A user can see the workflow widget on the record pages and the workflow record itself. |
+---------------------+----------------------------------------------------------------------------------------+
| Perform Transitions | A user can perform any transitions of the workflow.                                    |
+---------------------+----------------------------------------------------------------------------------------+

For each of this actions you can set an access level defining whether a user can view workflow / perform transitions for the records owned by the user themselves, or by the user's division, or for all records in the system, etc.

For more information about which access levels defines which range, see the `Access Levels <./access-management-access-levels>`__ topic.

.. image:: ../img/access_roles_management/roles_workflow_permissions1.png


Workflow Transition Permissions
"""""""""""""""""""""""""""""""

To enable a user to perform just certain transitions of a workflow and forbid others, set the permissions for each transition individually.

For more information about which access levels you can set, see the `Access Levels <./access-management-access-levels>`__ topic.

.. image:: ../img/access_roles_management/roles_workflow_permissions2.png


Assign and Combine Roles
^^^^^^^^^^^^^^^^^^^^^^^^^

Each OroCRM user must be assigned a role. A user can have several roles. This is a logical approach if we assume that roles may be based on job functions. For example, if you have roles 'Leads Development Representative' and 'Sales Representative' and some employees do both of these jobs, you simply assign them both roles instead of creating a specialized role that will cover the whole range of required permissions. 
For how to assign a role to a user, see the :ref:`Assign Roles While Creating a New User <user-guide-user-management-permissions>` section.


If a user has two or more roles with different permissions, in the result the user will have the maximum of rights granted by all of them.   

The following example shows what access level for an action on entity a user who is assigned two roles with different permissions will have:

+------------------------+----------------------------+----------------------------+
| Role 1                 | Role 2                     | Role 1 + Role 2            |
+========================+============================+============================+
| Entity: Account        | Entity: Account            | Entity: Account            |
|                        |                            |                            |  
| Action: View           | Action: View               | Action: View               |
|                        |                            |                            | 
| Access Level: **User** | Access Level: **Division** | Access Level: **Division** |
+------------------------+----------------------------+----------------------------+

Links
-----

For how a role is represented on the interface, see the :ref:`Roles on the Interface <user-guide-user-management-permissions-roles--interface>` topic.

For what actions you can perform with roles, see the :ref:`Actions with Roles <user-guide-user-management-permissions-roles--actions>` topic.

For examples on roles application, see the :ref:`Access Configuration Examples <user-guide-user-management-permissions-roles--examples>` topic.

.. include:: ../../img/buttons/include_images.rst
   :start-after: begin

## 1. INTRODUCTION

The lack of standards for RBAC has led to roles being implemented in different ways, impeding the advance of RBAC technology.

The goal of this paper is to provide a standard in this arena.

The basic role concept is simple: **establish permissions based on the functional roles in the enterprise, and then appropriately assign users to a role or set of roles**

Roles colud represent the tasks, responsibilities, and qualifications associated with an enterprise.

RBAC is a rich and open-ended concept which ranges from very simple at one extreme to fairly complex and sophisticated at the other. It has been recognized that a single definitive model for RBAC is therefore unrealistic.

**The NIST RBAC model is consequently organized in a four step sequence of increasing functional capabilities given below.**

- Flat RBAC
- Hierarchical RBAC
- Constrained RBAC
- Symmetric RBAC

<br>

## 2. MODEL OVERVIEW

![](../images/[DEV]%20The%20NIST%20Model%20for%20Role-Based%20Access%20Control:%20Towards%20A%20Unified%20Standard_07.png)

These levels are cummulative in that each includes the requirements of the previous ones in the sequence. **Each level adds exatly one new requirement.**

### 2.1 Flat RBAC

**Flat RBAC embodies the essential aspects of RBAC.**

The basic concept of RBAC is that users are assigned to roles, permissions are assignes to roles and users aquire permissions by being members of roles.

**The NIST RBAC model requires that `user-role assignment` and `permission-role assignment` can be many-to-many.**

**Flat RBAC requires that users can simultaneously exercise permissions of multiple roles.**

The features required of flat RBAC are obligatory for any form of RBAC and are almost obvious.

The main issue in defining flat RBAC is to determine which features to exclude.

**The NIST flat RBAC model has deliberately kept a very minimal set of features.**

In particular, these features accommodate traditional but robust group-based access control.

<br>

### 2.2 Hierarchical RBAC

**Hierarchical RBAC adds a requirement for supporting role hierarchies.**

A hierarchy is mathematically a partial order defining a seniority relation between roles, whereby senior roles acquire the permissions of their juniors.

The NIST model recognizes two sub-levels in this respect.

**1. General Hierarchical RBAC**

- In this case there is support for an arbitrary partial order to serve as the role hierarchy.

**2. Restricted Hierarchical RBAC**

- Some systems may impose restrictions on the role hierarchy. Most commonly, hierarchies are limited to simple structures such as trees or inverted trees.

<br>

### 2.3 Constrained RBAC

Constrained RBAC adds a requirement for enforcing separation of duties(SOD).

Many different SOD requirements have been identified in the literature.
- static SOD (based on user-role assignment)
- dynamic SOD (based on role activation)

<br>

### 2.4 Symmetric RBAC

Symmetric RBAC adds a requirement for permission-role review similar to user-role review introduced in level 1. Thus the roles to which a particular permissions is assigned can be determined as well as permissions assigned to a speciffic role.

<br>

## 3. FLAT RBAC

![](../images/[DEV]%20The%20NIST%20Model%20for%20Role-Based%20Access%20Control:%20Towards%20A%20Unified%20Standard_03.png)

> *" The requirement that users acquire permissions through roles is the essence of RBAC. "*


In Figure 1, A permission is an approval of a particular mode of access to one or more objects in the system.

**Permissions are always positive and confer the ability to the holder of the permission to perform some action(s) in the system.** :star:

**Flat RBAC requires that `UA(user-role assignment)`, `PA(permission-role assignment)` are many-to-many relations. This is an essential aspect of RBAC.** :star:

The concept of a session is not explicitly a part of flat RBAC.

**In some cases all roles of a user are activated in every session of the user. In other cases the user is given a choice to activate and deactivate roles in a given session at the user's discretion.** :star:

> 후자는 `AWS 계정 선택` 행위가 예시가 될 수 있겠다.

The NIST model dose not require support for sessions with discretionary role activation. **It does require the ability to activate multiple roles simultaneously and in a single session.**

**Flat RBAC requires support for user-role review whereby it can be effciently determined which roles a given user belongs to and which users a given role is assigned to.** :star:

The flat RBAC model leaves open many important issues that must be addressed in an implementation.

- There are no **scalability** requirements on the numbers of roles, users, permissions, etc.., that should supported.
- **The nature of permissions** and support for discretionary role activation is not fully specified.
- **Revocation** can occur when a user is removed from a role or a permission is removed from a role. How quickly the revocation actually takes place, particulary with respect to activity which is already under way, is left unspecified.
- The important issue of **role administration** is not specified. Role administration is concerned with who gets to assign users to roles and permissions to roles.

> 위 요소들을 다루지 않은 이유는 (1)모든 상황(벤더, 시장)에 적절한 기준이 아닐 수 있고, (2)충분한 협의가 이뤄지지 않았기 때문이다.

<br>

## 4. HIERARCHICAL RBAC

...

<br>

## 5. CONSTRAINED RBAC

...

<br>

## 6. SYMMETRIC RBAC

...

<br>

## 7. OTHER RBAC ATTRIBUTES

...

### 7.3 Negative permissions

**The NIST model is based on positive permissions** that confer the ability to do something on holders of the permission.

**But the NIST model does not rule out the use of so-called negative permissions which deny access.** **Thus vendors are free to add this feature.**

Nevertheless vendors and users are cautioned that use of negative permissions can be very confusing.

<br>

### 7.4 Nature of permissions :star::star:

**The nature of permissions is not specified in the NIST RBAC model.**

Permissions can be `fine-grained(e.g., at the level of individual objects)` or `coarse-grained(e.g., at the level of entire sub-systems)`.

**They can be defined in terms of primitive operations such as read and write, or abstract operations such as credit and debit.**

Permissions can also be customize.

**The exact nature of permissions is determined by the nature of the product.**

OS, DBMS, workflow systems, network management systems will all support different kinds of permissions. 

**Standardization of permissions is beyond the scope of a general-purpose access control model.**

<br>

### 7.6 Role engineering :star:

**The NIST RBAC model does not provide guidelines for designing roles** and assigning permissions and users to roles.

**This activity is called role engineering. Effective use of RBAC in large-scale is strongly dependent on effective role engineering.**

However, this issue is outside the scope of the NIST RBAC model.
## LDAP (Lightweight Directory Access Protocol)

LDAP is an open, vendor-neutral, industry standard application protocol that is used for accessing and maintaining distributed directory information services over an Internet Protocol (IP) network. It is predominantly employed for user authentication and authorization, as well as other directory-based services.

Main idea:

- A **directory** functions similarly to a database, but it primarily holds descriptive and attribute-based information rather than transactional data. It is structured in a hierarchical format, which typically organizes data into categories such as users, groups, and departments.
- Each **entry** within a directory serves as a unique record that represents an object, like a user, group, or computer. These entries are comprised of various attributes that define the characteristics and identity of the object being represented.
- An **attribute** is a specific data point associated with an entry that provides detailed information about that object. For example, attributes for a user might include an email address, full name, or password, all of which give context to the entry.
- The **schema** is essential for defining the structure of a directory’s data. It specifies what kinds of objects (such as users or devices) the directory can store, what attributes each object type can have, and the rules governing how these objects and attributes interact within the directory.

### LDAP Directory Structure

LDAP directories are hierarchical and follow a tree structure, similar to a file system.

**LDAP Directory Information Tree (DIT)**

```
                (Root)
                  |
         +--------+--------+
         |                 |
       dc=com           dc=org
         |                 |
    +----+----+            |
    |         |            |
  dc=example  dc=company   ...
    |
+---+---+
|       |
ou=People ou=Groups
   |          |
   |          +----------------+
   |                           |
+--+--+                    +---+---+
|     |                    |       |
uid=alice              cn=admins  cn=users
uid=bob
```

- **dc**: Domain Component (e.g., `dc=example,dc=com` represents `example.com`).
- **ou**: Organizational Unit (e.g., `ou=People`).
- **uid**: User ID (e.g., `uid=alice`).
- **cn**: Common Name (used for groups, e.g., `cn=admins`).

### LDAP Authentication Flow

```
[ Client Application ]
         |
         v
[ LDAP Library ]
         |
         v
[ LDAP Server ]
         |
         v
[ LDAP Directory (DIT) ]
```

**Steps:**

1. **Client Application**: Initiates a connection to authenticate a user.
2. **LDAP Library**: Handles communication with the LDAP server using the LDAP protocol.
3. **LDAP Server**: Processes the request and searches the DIT.
4. **LDAP Directory**: Retrieves the user entry and verifies credentials.

### Common LDAP Operations

1. **Bind**: Authenticates and specifies the LDAP protocol version. In simple terms, this operation logs a user into the LDAP server.

2. **Search & Compare**: These operations are used to search for and retrieve records from the LDAP database. Compare operation is used to compare a specified attribute value pair with an entry in LDAP server.

3. **Add, Delete & Modify**: These operations are used to create, delete, and update records in the directory.

4. **Unbind**: This operation terminates an LDAP session.

### LDAP Data Model

The LDAP data model is based on an "Entry" which is collection of attributes with a globally unique Distinguished Name (DN). The DN is used to refer an entry in the directory. Each of the entry's attributes has a type and one or more values. The types are typically mnemonic strings, like "cn" for common name, or "mail" for email address.

Example DN: "cn=John Doe,dc=example,dc=com"

### LDAP Search Filters

LDAP search filters enable the client to specify and control how the data should be searched. The filters are based on attributes in the entries. For example, you might request that the server return only users who are located in a certain city.

Example of a simple filter that will return all the entries that have the "objectclass" of "person" and "uid" of "jdoe": "(&(objectclass=person)(uid=jdoe))"

### LDAP Tools and Utilities

1. **ldapsearch**: A command-line utility that is used to search for entries in the LDAP directory. 

2. **ldapadd/ldapmodify**: These are used to add/modify entries in the LDAP server. 

3. **ldapdelete**: This command-line tool is used to delete entries from the LDAP directory.

4. **ldapwhoami**: It is used to return the DN (Distinguished Name) of the user/client as authenticated on the LDAP server.

5. **phpLDAPadmin, JXplorer, Apache Directory Studio**: These are examples of GUI-based LDAP browsers/managers, which can simplify interacting with the server.

## Implementing LDAP for Centralized Authentication Across Multiple Servers

Implementing LDAP for centralized authentication enables you to manage user credentials effectively across multiple servers. This setup not only provides consistent login information for users across all servers but also streamlines the administration of user accounts.

### Step-by-Step Guide to Implement LDAP Authentication

1. **Establish an LDAP Server**: Initially, you'll need to install and configure an LDAP server. For this guide, we'll use OpenLDAP, a widely-used LDAP server. Choose a dedicated server or one of your existing servers to host OpenLDAP.

Install the necessary OpenLDAP packages using the following commands:

```
sudo apt-get update
sudo apt-get install slapd ldap-utils
```

You'll then need to reconfigure the 'slapd' package to define your LDAP domain and assign an admin password:

```
sudo dpkg-reconfigure slapd
```

2. **Construct the Schema**: Define your directory's base structure, which includes organization and organizational unit entries. This is achieved by creating an LDIF (LDAP Data Interchange Format) file.

Here's an example LDIF file (`base.ldif`):

```
dn: dc=example,dc=com
objectClass: top
objectClass: dcObject
objectClass: organization
o: Example Company
dc: example

dn: ou=users,dc=example,dc=com
objectClass: top
objectClass: organizationalUnit
ou: users
```

Load this structure into your LDAP directory using the `ldapadd` command:

```
sudo ldapadd -x -D "cn=admin,dc=example,dc=com" -w your_admin_password -f base.ldif
```

3. **Populate the Directory**: Now, add entries for each user in your LDAP directory, incorporating necessary attributes like their credentials.

Here's an example user entry (`user.ldif`):

```
dn: uid=jdoe,ou=users,dc=example,dc=com
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
uid: jdoe
cn: John Doe
givenName: John
sn: Doe
mail: jdoe@example.com
userPassword: {CLEARTEXT}password
```

Load the user data into your LDAP directory:

```
sudo ldapadd -x -D "cn=admin,dc=example,dc=com" -w your_admin_password -f user.ldif
```

4. **LDAP Authentication Configuration on Servers**: Now, you'll need to install and set up the LDAP client on each server to authenticate users against your LDAP server.

Install the required packages:

```
sudo apt-get install libnss-ldap libpam-ldap nscd
```

The installation will prompt you to input your LDAP server URI, search base, and admin credentials. Enter the correct details to configure your LDAP client.

Additionally, make sure that your client is configured to establish a secure connection with your LDAP server. Modify the `/etc/ldap/ldap.conf` file to demand certificate verification:

```
TLS_REQCERT demand
```

5. **Verify Authentication**: Test that users can log into each server using their LDAP credentials. Additionally, verify that access control operates as expected, granting or denying access based on group membership or other attributes.

6. **Manage and Update the Directory**: As users are added, removed, or have their credentials altered, you'll need to update your LDAP directory accordingly. LDAP utilities like `ldapadd`, `ldapmodify`, and `ldapsearch` will help administrators manage user entries effectively.

## Example Scenario: Centralized Employee Directory

Let's consider Acme Corporation, a growing company with employees spread across multiple locations worldwide. They want a centralized system to store and manage the employee's contact information.

### Step 1: Setting Up LDAP Directory

First, they decide to deploy an LDAP server as a centralized employee directory. This directory will store data like first name, last name, email, phone number, and department details for each employee.

For this, they define an LDAP schema that outlines the format and type of information to be stored. The schema will define entries for `People` and `Groups`. In the `People` entries, they'll store individual user information. The `Groups` entries will contain department-based groups for each department in Acme Corporation.

You can find the IP address within the VM by running the following command in the VM's terminal:

```bash
ip addr show
```

or for Windows VMs:

```bash
ipconfig
```

### Step 2: Adding Employee Information

Next, Acme Corporation's HR team begins populating the LDAP directory with employee information. Each employee is represented as an `Entry` in the `People` category with attributes like:

```
dn: uid=jdoe,ou=people,dc=acme,dc=com
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
uid: jdoe
cn: John Doe
givenName: John
sn: Doe
mail: jdoe@acme.com
telephoneNumber: +1 123 456 7890
ou: Engineering
```

This entry provides a range of information about John Doe, a member of the Engineering department.

### Step 3: Interacting with the LDAP Directory

For employees to interact with this directory, they are provided with an LDAP client tool. They can use this tool to search the directory for their colleagues' contact information using their unique `uid` or their common name `cn`.

For instance, to find information about John Doe, an employee would execute a command similar to:

```
ldapsearch -x -H ldap://ldap.acme.com -b 'dc=acme,dc=com' '(uid=jdoe)'
```

This will return John Doe's contact information.

### Step 4: Managing the Directory

Acme Corporation appoints LDAP administrators who manage the directory. These administrators are responsible for adding new entries when new employees join, modifying existing entries when employees' details change, and deleting entries when employees leave the organization.

### Step 5: Utilizing LDAP for Authentication and Authorization

Beyond serving as an employee directory, Acme Corporation also uses their LDAP setup for authentication and authorization for their internal services. Employees can use their unique 'uid' and a corresponding password, which are stored securely in the LDAP directory, to log in to these services. Authorization is managed through `Group` entries that correspond to different access levels and departmental resources.

## Challenges

1. Install and configure an LDAP server on a Linux system. Set up the basic directory structure and include at least three organizational units (OUs).
2. Add entries to the LDAP directory, including users and groups. Practice creating at least 10 user entries and 3 groups, assigning users to different groups.
3. Configure a Linux system to use LDAP for user authentication. Test this by logging into the system with user credentials stored in the LDAP directory.
4. Develop a strategy for backing up the LDAP directory. Perform a backup, then restore from this backup to ensure the integrity and completeness of your backup method.
5. Use the `ldapsearch` command to perform various queries on the LDAP directory. Try to search for specific users, groups, and other entities based on different attributes.
6. Secure your LDAP communications with TLS/SSL. Configure the server for encrypted connections and verify the security by connecting to it with an LDAP client.
7. Choose an application or service (such as email or web service) that supports LDAP integration. Configure it to authenticate users against your LDAP directory.
8. Design and implement a custom LDAP schema for a specific use case (like managing inventory or tracking software licenses). Add attributes and object classes that are not available in the default schema.
9. Set up LDAP replication. Configure a secondary LDAP server and ensure that it synchronizes correctly with your primary LDAP server.
10. Simulate common LDAP connectivity issues and practice troubleshooting. Document each issue simulated, your diagnostic process, and the steps taken to resolve the issues.

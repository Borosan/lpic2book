# 210.3. LDAP client usage

**Weight**: 2

**Description:** Candidates should be able to perform queries and updates to an LDAP server. Also included is importing and adding items, as well as adding and managing users.

**Key Knowledge Areas:**

* LDAP utilities for data management and queries
* Change user passwords
* Querying the LDAP directory

**Terms and Utilities:**

* ldapsearch
* ldappasswd
* ldapadd
* ldapdelete

This course is about LDAP Client utilities and their usage. Obviously we need and OpenLDAP server up and running inorder to perform queries and do modifcations.

## We recommend you to study 210.4 course before start reading this course :-o

 **note**:LDAP client utilities usage might be different based on OpenLDAP server versions. To get the best results and covering LPIC2 exam objectives, we have used OpenLDAP v2.3.x on CentOS 5 in this course.

## ldapsearch

ldapsearch opens a connection to an LDAP server, binds, and performs a search using specified parameters. By default ldapsearch query local host computer for LDAP server but its is possible to run it from a remote client using -h option and specifying the server.

Now that some LDAP configuration has been added, lets check the results using ldapsearch command:

```text
[root@centos5-1 openldap]# ldapsearch -x -b '' -s base '(objectclass=*)' namingContexts
# extended LDIF
#
# LDAPv3
# base <> with scope baseObject
# filter: (objectclass=*)
# requesting: namingContexts
#

#
dn:
namingContexts: dc=example,dc=com

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

## ldapadd

The ldapadd command is an LDAP add-entry tool. Here we use ldapadd command which takes our ldif file definations and add it to our configuration:

Create OU inorder to give structure to our Directory Server, again we need to required ldif file:

```text
[root@centos5-1 openldap]# vi myou.ldif
[root@centos5-1 openldap]# cat myou.ldif 
dn: ou=managers,dc=example,dc=com
ou: managers
description : Managers in the company
objectclass: organizationalunit
```

and lets run ldapadd commands :

```text
[root@centos5-1 openldap]# ldapadd -x -D "cn=ldapadm,dc=example,dc=com" -W -f myou.ldif
Enter LDAP Password: 
adding new entry "ou=managers,dc=example,dc=com"
```

`-x` means use simple authentication,`-D`says that the admin account is setup here to adding things to our configuration, `-W` Prompt for simple authentication \(This is used instead of specifying the password on the command line\).`-f` specify ldif file.

Use slapcat command to see the results:

```text
[root@centos5-1 openldap]# slapcat
dn: dc=example,dc=com
dc: example
description:: Y3JlYXRpbmcgbXkgZGMg
objectClass: dcObject
objectClass: organization
o: example,com.
structuralObjectClass: organization
entryUUID: 4c4b3fa6-3ee8-1038-85fd-a183bee33ca9
creatorsName: cn=ldapadm,dc=example,dc=com
modifiersName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828083000Z
modifyTimestamp: 20180828083000Z
entryCSN: 20180828083000Z#000000#00#000000

dn: ou=managers,dc=example,dc=com
ou: managers
description: Managers in the company
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: 0a9d2e2a-3efc-1038-9390-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828105120Z
entryCSN: 20180828105120Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828105120Z
```

Now that we have "managers" ou we can add a record to it:

```text
[root@centos5-1 openldap]# vi myuser.ldif
[root@centos5-1 openldap]# cat myuser.ldif
dn: cn=Bob Smith,ou=managers,dc=example,dc=com
objectclass: inetOrgperson
cn: Bob Smith
cn: Bob J Smith
cn: bob smith
sn: smith
uid: bjsmith
userpassword: Aa12345
carlicense: abc123
homephone: 111-222-3344
mail: b.smith@example.com
mail: bsmith@example.com
mail: bob.smith@exmple.com
description: Big Boss
ou: IT Department
```

For more info about objectClass "inetOrgperson" try cat schema/inetorgperson.schema .

```text
[root@centos5-1 openldap]# ldapadd -x -D "cn=ldapadm,dc=example,dc=com" -W -f myuser.ldif 
Enter LDAP Password:
adding new entry "cn=Bob Smith,ou=managers,dc=example,dc=com"
```

and check:

```text
[root@centos5-1 openldap]# slapcat
dn: dc=example,dc=com
dc: example
description:: Y3JlYXRpbmcgbXkgZGMg
objectClass: dcObject
objectClass: organization
o: example,com.
structuralObjectClass: organization
entryUUID: 4c4b3fa6-3ee8-1038-85fd-a183bee33ca9
creatorsName: cn=ldapadm,dc=example,dc=com
modifiersName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828083000Z
modifyTimestamp: 20180828083000Z
entryCSN: 20180828083000Z#000000#00#000000

dn: ou=managers,dc=example,dc=com
ou: managers
description: Managers in the company
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: 0a9d2e2a-3efc-1038-9390-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828105120Z
entryCSN: 20180828105120Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828105120Z
```

lets add more users and OUs to our LDAP server:

```text
[root@centos5-1 openldap]# cat moreusers.ldif 
dn: cn=James Smith,ou=managers,dc=example,dc=com
objectclass: inetOrgPerson
cn: James Smith
cn: James J Smith
sn: James
uid: jsmith
userpassword: Aa12345
carlicense: A1B2C3
homephone: 222-333-4455
mail: j.smith@example.com
mail: jsmith@example.com
mail: james.smith@example.com
ou: managers

### add anothr Entry to our OU
dn: cn=Maria Garcia,ou=managers,dc=example,dc=com
objectclass: inetOrgPerson
cn: Maria Garcia
sn: garcia
uid: mgarcia
userpassword: Aa12345
carlicense: AABBCC
homephone: 333-444-4466
mail: m.garcia@example.com
mail: mgarcia@example.com
mail: maria.garcia@example.com
ou: managers
```

```text
[root@centos5-1 openldap]# ldapadd -x -D "cn=ldapadm,dc=example,dc=com" -W -f moreusers.ldif
Enter LDAP Password: 
adding new entry "cn=James Smith,ou=managers,dc=example,dc=com"

adding new entry "cn=Maria Garcia,ou=managers,dc=example,dc=com"
```

```text
[root@centos5-1 openldap]# cat moreou.ldif 
### Add Users OU
dn: ou=users,dc=example,dc=com
ou: users
description : Ordinary users in the company
objectclass: organizationalunit

### Add Devices OU

dn: ou=sales,dc=example,dc=com
ou: sales
description: Sales group OU
objectclass: organizationalunit
```

```text
[root@centos5-1 openldap]# ldapadd -x -D "cn=ldapadm,dc=example,dc=com" -W -f moreou.ldif
Enter LDAP Password: 
adding new entry "ou=users,dc=example,dc=com"

adding new entry "ou=sales,dc=example,dc=com"
```

and see current Directory Structure in our LDAP server:

```text
[root@centos5-1 openldap]# slapcat
dn: dc=example,dc=com
dc: example
description:: Y3JlYXRpbmcgbXkgZGMg
objectClass: dcObject
objectClass: organization
o: example,com.
structuralObjectClass: organization
entryUUID: 4c4b3fa6-3ee8-1038-85fd-a183bee33ca9
creatorsName: cn=ldapadm,dc=example,dc=com
modifiersName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828083000Z
modifyTimestamp: 20180828083000Z
entryCSN: 20180828083000Z#000000#00#000000

dn: ou=managers,dc=example,dc=com
ou: managers
description: Managers in the company
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: 0a9d2e2a-3efc-1038-9390-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828105120Z
entryCSN: 20180828105120Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828105120Z

dn: cn=James Smith,ou=managers,dc=example,dc=com
objectClass: inetOrgPerson
cn: James Smith
cn: James J Smith
sn: James
uid: jsmith
userPassword:: QWExMjM0NQ==
carLicense: A1B2C3
homePhone: 222-333-4455
mail: j.smith@example.com
mail: jsmith@example.com
mail: james.smith@example.com
ou: managers
structuralObjectClass: inetOrgPerson
entryUUID: 225aa230-3efd-1038-9391-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828105909Z
entryCSN: 20180828105909Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828105909Z

dn: cn=Maria Garcia,ou=managers,dc=example,dc=com
objectClass: inetOrgPerson
cn: Maria Garcia
sn: garcia
uid: mgarcia
userPassword:: QWExMjM0NQ==
carLicense: AABBCC
homePhone: 333-444-4466
mail: m.garcia@example.com
mail: mgarcia@example.com
mail: maria.garcia@example.com
ou: managers
structuralObjectClass: inetOrgPerson
entryUUID: 2279564e-3efd-1038-9392-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828105909Z
entryCSN: 20180828105909Z#000001#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828105909Z

dn: ou=users,dc=example,dc=com
ou: users
description: Ordinary users in the company
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: 66342080-3efd-1038-9393-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828110103Z
entryCSN: 20180828110103Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828110103Z

dn: ou=sales,dc=example,dc=com
ou: sales
description: Sales group OU
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: 6634f758-3efd-1038-9394-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828110103Z
entryCSN: 20180828110103Z#000001#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828110103Z
```

Check the result using ldapsearch :

```text
[root@centos5-1 openldap]# ldapsearch -x -b 'ou=managers,dc=example,dc=com' '(objectclass=inetorgperson)' uid
# extended LDIF
#
# LDAPv3
# base <ou=managers,dc=example,dc=com> with scope subtree
# filter: (objectclass=inetorgperson)
# requesting: uid 
#

# James Smith, managers, example.com
dn: cn=James Smith,ou=managers,dc=example,dc=com
uid: jsmith

# Maria Garcia, managers, example.com
dn: cn=Maria Garcia,ou=managers,dc=example,dc=com
uid: mgarcia

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
```

lets request for passwords to see wether it gives us or not:

```text
[root@centos5-1 openldap]# ldapsearch -x -b 'ou=managers,dc=example,dc=com' '(objectclass=inetorgperson)' password
# extended LDIF
#
# LDAPv3
# base <ou=managers,dc=example,dc=com> with scope subtree
# filter: (objectclass=inetorgperson)
# requesting: password 
#

# James Smith, managers, example.com
dn: cn=James Smith,ou=managers,dc=example,dc=com

# Maria Garcia, managers, example.com
dn: cn=Maria Garcia,ou=managers,dc=example,dc=com

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
```

it won't give us the password, no matter which user is performing this request. We can just see hashed passwords using slapcat command:

```text
[root@centos5-1 openldap]# slapcat
dn: dc=example,dc=com
dc: example
description:: Y3JlYXRpbmcgbXkgZGMg
objectClass: dcObject
objectClass: organization
o: example,com.
structuralObjectClass: organization
entryUUID: 4c4b3fa6-3ee8-1038-85fd-a183bee33ca9
creatorsName: cn=ldapadm,dc=example,dc=com
modifiersName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828083000Z
modifyTimestamp: 20180828083000Z
entryCSN: 20180828083000Z#000000#00#000000

dn: ou=managers,dc=example,dc=com
ou: managers
description: Managers in the company
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: 0a9d2e2a-3efc-1038-9390-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828105120Z
entryCSN: 20180828105120Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828105120Z

dn: cn=James Smith,ou=managers,dc=example,dc=com
objectClass: inetOrgPerson
cn: James Smith
cn: James J Smith
sn: James
uid: jsmith
userPassword:: QWExMjM0NQ==
carLicense: A1B2C3
homePhone: 222-333-4455
mail: j.smith@example.com
mail: jsmith@example.com
mail: james.smith@example.com
ou: managers
structuralObjectClass: inetOrgPerson
entryUUID: 225aa230-3efd-1038-9391-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828105909Z
entryCSN: 20180828105909Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828105909Z

dn: cn=Maria Garcia,ou=managers,dc=example,dc=com
objectClass: inetOrgPerson
cn: Maria Garcia
sn: garcia
uid: mgarcia
userPassword:: QWExMjM0NQ==
carLicense: AABBCC
homePhone: 333-444-4466
mail: m.garcia@example.com
mail: mgarcia@example.com
mail: maria.garcia@example.com
ou: managers
structuralObjectClass: inetOrgPerson
entryUUID: 2279564e-3efd-1038-9392-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828105909Z
entryCSN: 20180828105909Z#000001#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828105909Z

dn: ou=users,dc=example,dc=com
ou: users
description: Ordinary users in the company
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: 66342080-3efd-1038-9393-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828110103Z
entryCSN: 20180828110103Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828110103Z

dn: ou=sales,dc=example,dc=com
ou: sales
description: Sales group OU
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: 6634f758-3efd-1038-9394-2be1474ec233
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828110103Z
entryCSN: 20180828110103Z#000001#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828110103Z
```

another example of ldapsearch for searching for a specific user:

```text
[root@centos5-1 openldap]# ldapsearch -x -b 'ou=managers,dc=example,dc=com' '(cn=Maria Garcia)' uid
# extended LDIF
#
# LDAPv3
# base <ou=managers,dc=example,dc=com> with scope subtree
# filter: (cn=Maria Garcia)
# requesting: uid 
#

# Maria Garcia, managers, example.com
dn: cn=Maria Garcia,ou=managers,dc=example,dc=com
uid: mgarcia

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

But that is lots of information. By using `-L` switche Search results are display in LDAP Data Interchange Format detailed in ldif. A single `-L` restricts the output to LDIFv1. A second`-L` disables comments. A third`-L` disables printing of the LDIF version. The default is to use an extended version of LDIF.

```text
[root@centos5-1 openldap]# ldapsearch -L -x -b 'ou=managers,dc=example,dc=com' '(cn=Maria Garcia)' uid
version: 1

#
# LDAPv3
# base <ou=managers,dc=example,dc=com> with scope subtree
# filter: (cn=Maria Garcia)
# requesting: uid 
#

# Maria Garcia, managers, example.com
dn: cn=Maria Garcia,ou=managers,dc=example,dc=com
uid: mgarcia

# search result

# numResponses: 2
# numEntries: 1
```

```text
[root@centos5-1 openldap]# ldapsearch -LL -x -b 'ou=managers,dc=example,dc=com' '(cn=Maria Garcia)' uid
version: 1

dn: cn=Maria Garcia,ou=managers,dc=example,dc=com
uid: mgarcia
```

```text
[root@centos5-1 openldap]# ldapsearch -LLL -x -b 'ou=managers,dc=example,dc=com' '(cn=Maria Garcia)' uid
dn: cn=Maria Garcia,ou=managers,dc=example,dc=com
uid: mgarcia
```

### ldappasswd

We have used ldappasswd to generat hashed password for LDAP Admin user but we can use it to restart users passwords too:

```text
[root@centos5-1 openldap]# ldappasswd -x -D "cn=ldapadm,dc=example,dc=com" -s UserNewPassword -W "cn=Maria Garcia,ou=managers,dc=example,dc=com"
Enter LDAP Password: 
Result: Success (0)
```

`-x` for simple authentication , `-D` say this is the user that has the authorizationon on this particular directory server inorder to make these kind of changes, `-s` to set what ever password we like, -w for getting prompted for rootdn password, and finally secify who we are quering for. Now lets check th old password hash `QWExMjM0NQ==` with the new one:

```text
[root@centos5-1 openldap]# slapcat
dn: dc=example,dc=com
dc: example
description:: Y3JlYXRpbmcgbXkgZGMg
objectClass: dcObject
objectClass: organization
o: example,organization.
structuralObjectClass: organization
entryUUID: 99ad21b4-3eee-1038-99cf-79db8b518c63
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828091507Z
entryCSN: 20180828091507Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828091507Z

dn: ou=managers,dc=example,dc=com
ou: managers
description: Managers in the company
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: 78826438-3ef1-1038-99d0-79db8b518c63
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828093540Z
entryCSN: 20180828093540Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828093540Z

dn: cn=James Smith,ou=managers,dc=example,dc=com
objectClass: inetOrgPerson
cn: James Smith
cn: James J Smith
sn: James
uid: jsmith
userPassword:: QWExMjM0NQ==
carLicense: A1B2C3
homePhone: 222-333-4455
mail: j.smith@example.com
mail: jsmith@example.com
mail: james.smith@example.com
ou: managers
structuralObjectClass: inetOrgPerson
entryUUID: f565e3c8-3efe-1038-99d1-79db8b518c63
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828111213Z
entryCSN: 20180828111213Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828111213Z

dn: cn=Maria Garcia,ou=managers,dc=example,dc=com
objectClass: inetOrgPerson
cn: Maria Garcia
sn: garcia
uid: mgarcia
carLicense: AABBCC
homePhone: 333-444-4466
mail: m.garcia@example.com
mail: mgarcia@example.com
mail: maria.garcia@example.com
ou: managers
structuralObjectClass: inetOrgPerson
entryUUID: f56bc55e-3efe-1038-99d2-79db8b518c63
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828111213Z
userPassword:: e1NTSEF9NklNb0pSeVlHNTlEc0xOY2Zkanc2YUt3OSs3QnVNaUo=
entryCSN: 20180828125347Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828125347Z

dn: ou=users,dc=example,dc=com
ou: users
description: Ordinary users in the company
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: df4b1d00-3eff-1038-99d3-79db8b518c63
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828111845Z
entryCSN: 20180828111845Z#000000#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828111845Z

dn: ou=sales,dc=example,dc=com
ou: sales
description: Sales group OU
objectClass: organizationalUnit
structuralObjectClass: organizationalUnit
entryUUID: df4c13ea-3eff-1038-99d4-79db8b518c63
creatorsName: cn=ldapadm,dc=example,dc=com
createTimestamp: 20180828111845Z
entryCSN: 20180828111845Z#000001#00#000000
modifiersName: cn=ldapadm,dc=example,dc=com
modifyTimestamp: 20180828111845Z
```

To avoid setting password in the command use `-S` option:

```text
[root@centos5-1 openldap]# ldappasswd -x -D "cn=ldapadm,dc=example,dc=com" -S -W "cn=Maria Garcia,ou=managers,dc=example,dc=com"
New password: 
Re-enter new password: 
Enter LDAP Password: 
Result: Success (0)
```

and check the previous password hash with the new one using slapcat command.

### ldapdelete

lapdelete using similar information like ldapadd, ldapsearch and ldappasswd. It allows us to delete a record. Lets delete "Maria Garcia"

```text
[root@centos5-1 openldap]# ldapdelete "cn= Maria Garcia,ou=managers,dc=example,dc=com" -x -D "cn=ldapadm,dc=example,dc=com" -W
Enter LDAP Password:
```

to make sure that the record has been deleted try to delete it again:

```text
[root@centos5-1 openldap]# ldapdelete "cn= Maria Garcia,ou=managers,dc=example,dc=com" -x -D "cn=ldapadm,dc=example,dc=com" -W
Enter LDAP Password: 
ldap_delete: No such object (32)
        matched DN: ou=managers,dc=example,dc=com
```

and it will bark that the object does not exist.

Now we are able to use lapadd, lapsearch and lapdelete both on the client and the server.


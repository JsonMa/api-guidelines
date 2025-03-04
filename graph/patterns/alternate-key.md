# Alternate Key Pattern

Microsoft Graph API Design Pattern

_The Alternate Key Pattern provides the ability to query for a single, specific resource identifiable through an alternative set of properties that is not its primary key_

## Problem

---

The resources exposed in Graph are identified through a Primary Key - which guarantees uniqueness inside the same resource type. Often though, that same resource can also be uniquely identified by an alternative, more convenient property (or set of properties) that provides a better developer experience.

Take a look at the `user` resource: while the `id` remains a perfectly valid way to get the resource details, the `mail` address is also an unique property that could be used to identify it.

While it is still possible to use the `$filter` query parameter, such as

`GET https://graph.microsoft.com/v1.0/users?$filter=mail eq 'bob@contoso.com'`, the returned result is wrapped in an array that needs to be unpacked.

## Solution

---

Resource addressing via an alternative key can be achieved using the same parentheses-style convention as for the canonical key, with one difference: single-part alternate keys MUST specify the key property name to unambiguously determine the alternate key. (Note: this is a hypothetical sample)

https://graph.microsoft.com/v1.0/users(0) - Retrieves the employee with ID = 0
https://graph.microsoft.com/v1.0/users(email='bob@contoso.com') Retrieves the employee with the email matching `bob@contoso.com`

## When to Use this Pattern

---

This pattern works and makes sense when the alternate key is good enough to identify a single resource and provides an useful alternative to the client.

## Example

---

The same user identified via the alternate key SSN, the canonical (primary) key ID using the non-canonical long form with specified key property name, and the canonical short form without key property name

Declare `mail` and `ssn` as alternate keys on an entity:

```xml
<EntityType Name="user">
   <Key>
     <PropertyRef Name="id" />
   </Key>
   <Property Name="id" Type="Edm.Int32" />

   <Property Name="mail" Type="Edm.String" />
   <Property Name="ssn" Type="Edm.String" />
   <Annotation Term="Keys.AlternateKeys">
      <Collection>
         <Record>
            <PropertyValue Property="Key">
               <Collection>
                  <Record>
                     <PropertyValue Property="Name" PropertyPath="mail" />
                  </Record>
               </Collection>
            </PropertyValue>
         </Record>
         <Record>
            <PropertyValue Property="Key">
               <Collection>
                  <Record>
                     <PropertyValue Property="Name" PropertyPath="ssn" />
                  </Record>
               </Collection>
            </PropertyValue>
         </Record>
      </Collection>
   </Annotation>
</EntityType>
```

1. Get a specific resource through `$filter`:

```http
GET https://graph.microsoft.com/v1.0/users/?$filter=ssn eq '123-45-6789'
```

```json
{
  "value": [
    {
      "givenName": "Bob",
      "jobTitle": "Retail Manager",
      "mail": "bob@contoso.com",
      "mobilePhone": "+1 425 555 0109",
      "officeLocation": "18/2111",
      "preferredLanguage": "en-US",
      "surname": "Vance",
      "userPrincipalName": "bob@contoso.com",
      "id": "1a89ade6-9f59-4fea-a139-23f84e3aef66"
    }
  ]
}
```

2. Get a specific resource either through its primary key, or through the two alternate keys:

```http
GET https://graph.microsoft.com/v1.0/users/1a89ade6-9f59-4fea-a139-23f84e3aef66
GET https://graph.microsoft.com/v1.0/users(ssn='123-45-6789')
GET https://graph.microsoft.com/v1.0/users(mail='bob@contoso.com')
```

**NOTE:** When requesting a resource through its primary key you might want to prefer to use key-as-segment (as shown above). Also, the key-as-segment does not work for alternate keys.

All of the 3 will yield the sare response:

```json
{
  "givenName": "Bob",
  "jobTitle": "Retail Manager",
  "mail": "bob@contoso.com",
  "mobilePhone": "+1 425 555 0109",
  "officeLocation": "18/2111",
  "preferredLanguage": "en-US",
  "ssn": "123-45-6789",
  "surname": "Vance",
  "userPrincipalName": "bob@contoso.com",
  "id": "1a89ade6-9f59-4fea-a139-23f84e3aef66"
}
```

3. Requesting a resource for an unsupported alternate key property

```http
GET https://graph.microsoft.com/v1.0/users(name='Bob')

400 Bad Request
```

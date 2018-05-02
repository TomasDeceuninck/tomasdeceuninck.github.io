---
layout: post
title:  "Create Folder Share"
subtitle: "and set the permissions"
date:   2018-05-02
tags: PowerShell
background: '/img/posts/default.jpg'
comments: true
---
<a id="top"></a>

[Top]: #top

When I was moving a share with some specific permissions I was reminded why you should script configurations. Documenting specific permissions configurations can be cumbersome and it is always easy to overlook a specific setting.

So, lets analyze the current configuration and create a PowerShell script to configure the new share:

- [Analyze Current Permissions](#analyze-current-permissions)
- [Create New Folder](#create-new-folder)
- [Set Permissions](#set-permissions)
- [Share Folder](#share-folder)

**TL;DR**: [This script](#full-script) creates a folder, sets the permissions, and shares it.

## Analyze Current Permissions

When analyzing permissions `Get-ACL` is your friend.

Go to the folder you want to analyze and get the current ACL (Access Control List):

```powershell
Get-ACL . | % Access
```

This will give you a list of several Access Rules. For example:

```
FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : NT AUTHORITY\SYSTEM
IsInherited       : False
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None
```

Based on these rules we will be able to create Access Rules to add to the ACL of our new folder.

[Top]

## Create New Folder

Not that hard, but FYI:

```powershell
$folder = New-Item -Path $folderName -ItemType Directory
```

[Top]

## Set Permissions

When setting the permissions there are 2 options:

- Create new ACL from scratch
- Get current ACL and add/remove what you want

In this example I will be doing the latter.

First you get the ACL of your newly created folder:

```powershell
$acl = Get-ACL $folder
```

Next we want to add extra Access Rules. These objects can be created in two different ways, In my example I use both.

1. Create Access Rule using **New-Object**

    Based on your analysis you can create a new `System.Security.AccessControl.FileSystemAccessRule` object.

    ```powershell
    $acl.AddAccessRule(
    (New-Object -TypeName System.Security.AccessControl.FileSystemAccessRule -ArgumentList @(
        (New-Object System.Security.Principal.NTAccount($group.Name)),
        [System.Security.AccessControl.FileSystemRights]'CreateFiles, WriteExtendedAttributes, WriteAttributes, ReadAndExecute, Synchronize',
        [System.Security.AccessControl.InheritanceFlags]'ContainerInherit, ObjectInherit',
        [System.Security.AccessControl.PropagationFlags]::None,
        [System.Security.AccessControl.AccessControlType]::Allow
    ))
    )
    ```

2. Create Access Rule using **AccessRuleFactory**

    Sometimes when you analyze the ACL of an object you get a number as `FileSystemRights`.

    > To work out what these permissions actually are, you need to look at  which bits are set when you treat that number as 32 separate bits  rather than as an Integer (as Integers are 32 bits long), and compare  them to this diagram:   [http://msdn.microsoft.com/en-us/library/aa374896(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/aa374896(v=vs.85).aspx){:target="_blank"}
    > 
    > quote from [this](https://social.technet.microsoft.com/Forums/en-US/cb822c55-9f96-48e6-9c60-ca64ed13ef94/what-is-the-diference-between-acl-access-rule-268435456-and-fullcontrol?forum=winserverpowershell){:target="_blank"} forum post by Yan Li

    When you know what it does and just want to create an Access Rule that gives those specific permissions you can use the `AccessRuleFactory` function of an ACL.

    ```powershell
    $acl.AddAccessRule(
    ($acl.AccessRuleFactory(
        (New-Object System.Security.Principal.NTAccount('CREATOR OWNER')),
        268435456,
        $false,
        'ContainerInherit, ObjectInherit',
        'InheritOnly',
        'Allow'
    ))
    )
    ```

Once we added all Access Rules to the `$acl` parameter we can set the ACL of our folder.

```powershell
Set-ACL -Path $folder -ACLObject $acl
```

[Top]

## Share Folder

Now that the permissions are set you can share the folder. Share permissions will be set to 'Everyone' since the NTFS permissions will restrict access.

```powershell
New-SmbShare -Name $shareName -Path $folder.FullName -FullAccess 'Everyone'
```

[Top]

## Full Script

<script src="https://gist.github.com/TomasDeceuninck/003d4ddb124e8fc89212d78c4a44e5f7.js"></script>

[Top]

## Testing Administrative SysAdmin Permissions

After testing the standard SysAdmin role, I moved on to the privileged Administrative SysAdmin role.

The purpose of this testing is to confirm that the Administrative SysAdmin accounts can perform the higher-risk operations intentionally denied to standard SysAdmins. This page will be expanded as I complete each test.

### User Objects

#### Step 1: Create a User Object

I first attempted to create a test user in the Company Users OU:
```Powershell
New-ADUser -Name "Test User" `
    -SamAccountName "tuse" `
    -Path "OU=Company Users,DC=earth,DC=local" `
    -Enabled $true `
    -AccountPassword (ConvertTo-SecureString "Password1!" -AsPlainText -Force)
```
The command returned no error.

I verified that the user had been created successfully:
```Powershell
Get-ADUser -Identity "tuse"
```
![New-ADUser](/assets/images/adm_sys_test_user.png)

The query returned the new account, confirming that the Administrative SysAdmin role could create user objects in Company Users.

However, I had not supplied GivenName or Surname, so those attributes were not populated.

I created a second account with the additional attributes:
```Powershell
New-ADUser -Name "Test2 User2" `
    -GivenName "Test2" `
    -Surname "User2" `
    -SamAccountName "tuse2" `
    -Path "OU=Company Users,DC=earth,DC=local" `
    -Enabled $true `
    -AccountPassword (ConvertTo-SecureString "Password1!" -AsPlainText -Force) `
    -Verbose
```
I then verified that tuse2 had been created.

!!! success "User creation confirmed"
The Administrative SysAdmin role could create user objects in the delegated Company Users scope.

!!! note "Name attributes"
The Name parameter creates the object's display name but does not automatically populate GivenName and Surname. Those attributes must be supplied separately when required.

#### Step 2: Delete a User Object

I next attempted to delete the second test account:
```Powershell
Remove-ADUser -Identity "tuse2"
```
The deletion initially returned Access is denied.

I reviewed Effective Access for the Administrative SysAdmin account. The interface showed permission to create and delete user objects.

I had encountered a similar discrepancy while testing Write all properties for the standard SysAdmin role. My first troubleshooting step was therefore to use the Delegation of Control Wizard and select the relevant predefined task rather than relying on the earlier custom permission selection.

After applying the predefined delegation, I reran the deletion command. It completed without an error.

I confirmed that the account no longer existed:
```Powershell
Get-ADUser -Identity "tuse2"
```
The query could no longer find the user, validating the deletion.

![Delete Test User](/assets/images/adm_sys_remove_test_user.png)

!!! success "User deletion confirmed"
After correcting the delegation, the Administrative SysAdmin role could delete user objects within the intended scope.

!!! tip "Validate effective permissions operationally"
Effective Access is useful for investigation, but the final validation should be the real administrative operation. The initial deletion test exposed a difference between the displayed permissions and the permission set required by the command.

#### Step 3: Modify User Attributes

I tested whether the Administrative SysAdmin role could modify user attributes by updating the remaining test account:
```Powershell
Set-ADUser -Identity "tuse" `
    -Department "Finance" `
    -Title "Test Man" `
    -Description "Angela"
```
The command completed successfully.

I verified the updated attributes:
```Powershell
Get-ADUser -Identity "tuse" `
    -Properties Department, Title, Description
```
The query returned the new department, title and description values.

!!! success "User-property modification confirmed"
The delegated Write all properties permission allowed the Administrative SysAdmin role to modify user attributes in Company Users.

#### Step 4: Modify Group Membership

I attempted to add the test user to EL_HelpDesk:
```Powershell
Add-ADGroupMember -Identity "EL_HelpDesk" `
    -Members "tuse"
```
The command initially returned:

<div class="error-message">
Insufficient access rights to perform the operation.
</div>

The account had broad permissions over Company Users, but the error identified EL_HelpDesk as the target ADGroup. This directed the investigation towards the permissions on the "Groups" OU containing the security groups.

I reviewed the Groups OU and found delegated entries for the standard SysAdmin role, but none for the Administrative SysAdmin role.

Permissions on Company Users did not extend to group objects stored in a separate OU. I therefore granted the Administrative SysAdmin role the intended management permissions over the Security Groups OU.

After applying the change, I reran Add-ADGroupMember and it completed successfully.

I verified the membership through the test user's MemberOf attribute:
```Powershell
Get-ADUser -Identity "tuse" -Properties MemberOf |
    Select-Object Name, MemberOf
```
![Modify Group validation](/assets/images/adm_sys_modify_group_validation.png)

!!! success "Group-membership management confirmed"
The Administrative SysAdmin role could manage group membership after permissions were applied to the OU containing the group objects.

!!! info "Permission scope matters"
Permission to modify a user object does not automatically grant permission to modify a group object. Add-ADGroupMember changes the target group's membership attribute, so the required delegation must apply to the group and its OU.

!!! note "Interpreting denial messages"
Access is denied and Insufficient access rights should not be treated as reliable measurements of how close an account is to being authorised. The wording can vary between tools and APIs. The target object, requested operation and applicable ACL provide stronger diagnostic evidence.

---

### Computer Objects

#### Step 1: Create Computer Objects

I next tested whether my Administrative SysAdmin account could create computer objects in the Workstations OU.

I initially ran:

```powershell
New-ADComputer `
    -Name "TEST-PC01" `
    -SamAccountName "TEST-PC01$" `
    -Path "OU=Workstations,DC=earth,DC=local" `
    -Description "Administrative SysAdmin delegation test" `
    -Enabled $true `
    -Verbose
```

The command reached the correct target location but failed.

!!! failure "Computer-object creation failed"

```text
VERBOSE: Performing the operation "New" on target
"CN=TEST-PC01,OU=Workstations,DC=earth,DC=local".

New-ADComputer : A required attribute is missing
At line:1 char:1
+ New-ADComputer
+ ~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified:
                              (CN=TEST-PC01,OU...=earth,DC=local:String)
                              [New-ADComputer], ADException
    + FullyQualifiedErrorId : ActiveDirectoryServer:8316,
                              Microsoft.ActiveDirectory.Management.Commands.NewADComputer
```

##### Isolating the Cause

To determine whether one of the optional parameters was causing the error, I repeated the test using a minimal command:

```Powershell
New-ADComputer `
    -Name "TEST-PC01" `
    -Path "OU=Workstations,DC=earth,DC=local" `
    -Verbose
```

The minimal command produced the same error. This confirmed that the problem was not caused by the additional attributes in the original command.

##### Reviewing the Delegated Permissions

I reviewed the permissions assigned to EL_Adm_SysAdmins on the Workstations OU.

The Administrative SysAdmin permissions had previously been configured for the Company Users and security-groups OUs. However, the required computer-object permissions had not been correctly delegated on the Workstations OU.

I corrected the delegation by granting EL_Adm_SysAdmins the following permissions over computer objects within the Workstations OU:

* Create computer objects
* Delete computer objects
* Read all properties
* Write all properties
* Reset password
* Validated write to DNS host name
* Validated write to service principal name


##### Retesting Computer-Object Creation

After correcting the delegation, I repeated the computer-object creation command:

```Powershell
New-ADComputer `
    -Name "TEST-PC01" `
    -Path "OU=Workstations,DC=earth,DC=local" `
    -Description "Administrative SysAdmin delegation test" `
    -Enabled $true `
    -Verbose
```
!!! success "Computer object created successfully"

This time, the command completed successfully without producing an error.

I verified that the new computer object existed in the expected OU:
```Powershell
Get-ADComputer -Identity "TEST-PC01" `
    -Properties Description |
    Select-Object Name, Enabled, Description, DistinguishedName
```
The query returned TEST-PC01 under:

OU=Workstations,DC=earth,DC=local

![success full object creation](/assets/images/adm_sysadmin_New_ADComp.png)

#### Step 2: Delete computer objects:



#### Step 3: Reset computer account password:



#### Step 4: Write all properties on computer objects:

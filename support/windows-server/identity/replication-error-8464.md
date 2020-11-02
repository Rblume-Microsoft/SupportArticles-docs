---
title: Active Directory replication error 8464
description: Describes the symptoms, cause, and resolution for resolving issues in which Active Directory replication fails and returns error 8464.
ms.date: 09/10/2020
author: Deland-Han
ms.author: delhan
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika, justintu
ms.prod-support-area-path: Active Directory replication
ms.technology: ActiveDirectory  
---
# Active Directory replication error 8464: Synchronization attempt failed

This article helps you troubleshoot the Active Directory replication error 8464.

_Original product version:_ &nbsp; Windows Server 2019, Windows Server 2016, Windows Server 2012 R2  
_Original KB number:_ &nbsp; 3001248

## Notice

**Home users:** This article is only intended for technical support agents and IT professionals. If you're looking for help with a problem, [ask the Microsoft Community](https://answers.microsoft.com).

## Symptoms

This article describes the symptoms, cause, and resolution for resolving issues in which Active Directory replication fails and returns error 8464:

|Hex error code|0x00002110|
|---|---|
|Dec error code|8464|
|Symbolic Name|ERROR_DS_DRA_INCOMPATIBLE_PARTIAL_SET|
|Error Description|Synchronization attempt failed because the destination DC is currently waiting to synchronize new partial attributes from source. This condition is normal if a recent schema change modified the partial attribute set. The destination partial attribute set is not a subset of source partial attribute set.    |
|Header|winerror.h|
|||

Additionally, an ActiveDirectory_DomainService or NTDS Replication event 1704 entry that resembles the following is logged:

> Log Name            Directory Service  
Event ID               1704  
Event Source      ActiveDirectory_DomainService / NTDS Replication  
Task Category    Global Catalog  
Event Text           The global catalog initiated replication of a member of the partial attribute set for the following directory partition from the following domain controller.  
Directory partition:  
DC=\<name>,DC=\<domain>,DC=com  
Domain controller:  
\<fqdn>  
This is a special replication cycle due to the addition of one or more attributes to the partial attribute set.

**Note** It is typical to see Active Directory replication status 8464 after you extend the schema or after you add new attributes to the partial attribute set (PAS). This message states that replication is delayed temporarily. Active Directory replication status 8464 goes away after the PAS finishes the update process.

## Cause

This issue occurs because PAS synchronization is triggered when an attribute is added to the PAS.

For the details, see the More Information section.

## Resolution

Because this is a standard part of PAS synchronization, resolution steps are not needed. See the More Information section for troubleshooting steps if this replication status persists in the environment for more than a week. 

## More information  

The details of the cause for status 8464 

The Schema definition for an attribute is stored in the Schema partition as an attributeSchema object. Checking the Replicate this attribute to the Global Catalog check box sets the isMemberOfPartialAttributeSet attribute to TRUE on the attributeSchema object. Any attributeSchema object that has this attribute set to TRUE will cause the corresponding attribute to be included in the Partial Attribute Set.

When PAS synchronization occurs (that results from PAS extension), there is a specialized task in the replication task queue. The DRS_SYNC_PAS flag identifies this specialized task.

An unoptimized Active Directory topology or Active Directory replication failures may result in a significant delay in the PAS update process. It is typical to see Active Directory replication status 8464 during the PAS update process.  
 How to check Active Directory replication status 8464 

The Repadmin commands and other tools that provide an Active Directory replication status report state that a replication attempt is delayed with status 8464.
The following is the Repadmin commands and other tools that typically cite the 8464 status, including but are not limited to:

- REPADMIN /SHOWREPL 
- REPADMIN /REPLSUM 
- REPADMIN /REPLICATE 
- [Active Directory Replication Status Tool](https://www.microsoft.com/download/details.aspx?id=30005)

The following is a sample output from "Repadmin /showrepl" that shows incoming replication from DC2 to DC1 being delayed:

```console
 Domain\DC2 DSA Options: IS_GC Site Options: (none) DSA object GUID: <GUID> DSA invocationID: <ID> DC=child,DC=root,DC=contoso,DC=com Domain\DC1 via RPC DSA object GUID: <GUID> Last attempt @ 2014-08-28 04:50:44 was delayed for a normal reason, result 8464 (0x2110)
```

The following is the verbose output of the Repadmin /showrepl command:

```console
Domain\TRDC1 via RPC DSA object GUID: <GUID> Address: xxxxxxxxxx._msdcs.root.contoso.com DSA invocationID: <ID> SYNC_ON_STARTUP DO_SCHEDULED_SYNCS PARTIAL_ATTRIBUTE_SET USNs: 0/OU, 234943/PU Last attempt @ <Date & Time> was delayed for a normal reason, result 8464 (0x2110): Synchronization attempt failed because the destination DC is currently waiting to synchronize new partial attributes from source. This condition is normal if a recent schema change modified the partial attribute set. The destination partial attribute set is not a subset of source partial attribute set. Last success @ <Date & Time>.
```  

How to determine the destination domain controller 

> [!NOTE]
> These steps require an understanding of the environment's Active Directory replication topology, correlation of replication status data and temporary modification of Active Directory replication interval or connections. 

1. Identify one destination domain controller for one partition logged Active Directory replication status 8464. Use this domain controller and partition for these steps (do not jump around between partitions and domain controllers).

    > [!NOTE]
    > This step makes you to focus on updating the bridgehead servers and hub-site domain controller first. 
2. Collect the following data. Check replication status results for the destination domain controller and source domain controller. Run the following commands to export the results: 
   1. Compare the current PAS synchronization state among all global catalog servers. Run the following command to export the result to a pas_domain.txt file:

    ```console
    repadmin /showattr gc: <Partition_DN> /gc /atts:partialattributeset >pas_domain.txt
    ```  

2. Check replication status results for the destination and source domain controllers. Run the following commands to export the results:

```console
Repadmin /showrepl <DestinationDC> /verbose >repl_destDC.txt
```  

```console
Repadmin /showrepl <SourceDC> /verbose >repl_sourceDC.txt
```  

```console
Repadmin /showrepl * /csv >showrepl.csv
```  

3. List of all attributes in the PAS. This is useful for determining the current count. Run the following command to export the result to a pas.txt file:

```console
repadmin /showattr fsmo_schema: ncobj:schema: /filter:"(ismemberofpartialattributeset=TRUE)" /subtree /atts:dn >pas.txt
```  

4. Check event ID 1704 and 1702 as they indicate the PAS synchronization is complete in the Directory Service event log. Here is an example of event ID 1702:

> Log Name            Directory Service  
Event ID               1702  
Event Source      ActiveDirectory_DomainService / NTDS Replication  
Task Category    Global Catalog  
Event Text           The global catalog completed synchronization of the partial attribute set for the following directory partition from the following domain controller.  
Directory partition:  
DC=\<name>,DC=\<domain>,DC=com  
Domain controller:  
\<fqdn>  
This is a special replication cycle due to the addition of one or more attributes to the partial attribute set.

3. Analyse the data based on the destination domain controller with outdated PAS or source domain controller with outdated PAS. 
   - If the destination domain controller does not have the updated PAS, do the following: 
    1. Determine whether any source partners have the updated value. 
    2. Update destination and any source domain controllers that are out of date to clear the status 8464. 
    3. Manually start replication with source domain controllers that are up to date. Or, create and replicate source domain controllers if connections do not exist. 
    4. When the destination domain controller is updated, status 8464 will be logged for any source domain controllers that are not updated. 
   - If the destination domain controller has the updated PAS, but the source domain controller does not, the status 8464 will not clear until the source is updated. Or, you can update the source domain controller by manually starting replication with a domain controller that is up to date.   
 Pas_domain.txt instructions 

The value of interest in the output is listed after the v1.cAttrs =  text. This numeric value displays how many attributes are included in the PAS. Compare these values among each global catalog (GC) for each partition. If all GCs display the same value, all GCs are in-sync (they either all have the updated PAS, or they all do not). If all values are the same, compare them with the values from output in other partitions or the dump schema, and count the list of attributes in the PAS.

The following is a sample log, where DC1 has not updated the partial attribute set for the CHILD partition. Also DC2 has completed the PAS update process. No data is logged for ChildDC1, because the partialattributeset attribute has no data due to ChildDC1 containing a full copy of the Child domain partition.

> Repadmin: running command /showattr against full DC DC1.root.contoso.com DN: DC=child,DC=root,DC=contoso,DC=com 1> partialAttributeSet: { dwVersion = 1; dwFlag = 0; V1.cAttrs = 196, V1.rgPartialAttr = 0, 3, 4, 6, 7, 8, 9 Repadmin: running command /showattr against full DC ChildDC1.child.root.contoso.com DN: DC=child,DC=root,DC=contoso,DC=com Repadmin: running command /showattr against full DC DC2.root.contoso.com DN: DC=child,DC=root,DC=contoso,DC=com 1> partialAttributeSet: { dwVersion = 1; dwFlag = 0; V1.cAttrs = 203, V1.rgPartialAttr = 0, 3, 4, 6, 7, 8, 9

Pas.txt Instructions 

#### Identify the list of attributes in the PAS 

To see a list of the attributes in the PAS, use repadmin or another tool to query the Schema partition for all attributes where the ismemberofpartialattributeset value is set to TRUE:

```console
repadmin /showattr fsmo_schema: ncobj:schema: /filter:"(ismemberofpartialattributeset=TRUE)" /subtree /atts:dn >pas.txt
```  

Make sure that the word TRUE is in all uppercase text.

You can also use LDIFDE to achieve this data together with the count:

```console
Ldifde -f pas.txt -d "cn=schema,cn=configuration,dc=forestRootDN" -r "(ismemberofpartialattributeset=TRUE)" 196 entries exported
```  

#### Identify the count of attributes in the PAS 

To achieve a count of the attributes from the repadmin output, follow these steps: 
1. Open the text file in Notepad. 
2. Delete any blank lines at the beginning and end of the file. 
3. Delete the line at the top of the file that begins with "Repadmin: running command /showatt." 
4. Put your pointer on the last line of text in the file, then press the Ctrl + G keyboard shortcut to open up the **Go To Line**  dialog box. The line number in this window represents the count for attributes in the partial attribute set.   
 Directory Service event log 

Enable diagnostic logging for global catalog events in order to view additional detail about the partial attribute set update cycle. After enabling replication event verbosity, view the Directory Services event log.

To enable diagnostic logging for global catalog events, follow these steps: 
1. Open Regedit. 
2. Locate and then click the following registry key: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NTDS\Diagnostics` 
3. Configure event logging for global catalog: 
   1. On the right side of Registry Editor, double-click the **18 Global Catalog** entry. 
   2. Type 3 in the Value data box, and then click **OK**. 
4. Close Regedit.  View the following events in the Directory Service event log: 

Replication status cycle in partial attribute synchronization 

The Active Directory replication status 8464 message is logged when the destination domain controller is waiting to synchronize the updated PAS from the source domain controller.

> [!NOTE]
> The domain controller that is selected for the PAS_Sync task can move to a different source domain controller on the next replication interval (from the existing set of replica partners).

New attempts to synchronize the PAS are based on the replication schedule. When a different source is selected for the PAS_Sync task, replication will proceed usually with the prior source domain controller. After the PAS is successfully updated, replication for the same partition to the domain controller that is updated will fail from domain controllers without the updated PAS. The same replication status is logged for this scenario.

When the destination domain controller has not updated the PAS, one of the following processes occurs: 
- Selects one replica source to update PAS. Then, event 1704 is logged when the sync begins. 
- If source does not have the updated PAS itself, Active Directory replication status 8464 is logged, and event ID 1705 is logged if diagnostic logging is enabled. 
- If the PAS update task failed, and then a new source is selected, event ID 1706 is logged if diagnostic logging is enabled. 
- Replication from other domain controllers for the same partition proceeds as usual (status 0 is logged if there are no failures). 

#### Example of the PAS synchronization cycle 

This section is a sample of the PAS synchronization cycle. The following table is the domain controllers in the forest: 

| Domain Controllers| Domain |
|---|---|
| DC1| `Root.contoso.com` |
| DC2| `Root.contoso.com` |
| ChildDC1| `Child.root.contoso.com` |
| ChildDC2| `Child.root.contoso.com` |
| TRDC1| `Treeroot.fabrikam.com` |
|||

The following is the structure of the forest:

:::image type="content" source="./media/replication-error-8464/foreast-structure.png" alt-text="The structure of the forest.":::

Consider the following scenario:

- 7 new attributes are added to the PAS by using Schema extension. Therefore, the count for attributes in the PAS increases from 196 to 203. 
- This starts PAS synchronization. All GCs must now source the data for these seven attributes in each GC partition. 
- This diagram shows the update of just one partition. 
- The replication interval in this environment is 15 minutes. 
- A pre-existing condition exists that blocks replication from a DC hosting a writable copy of the partition.

In this scenario, the follow processes occur: 

1. When destination domain controller has not updated the PAS: 
   1. DC1 selects DC2 for PAS_SYNC. Because DC2 also has the old PAS, Active Directory replication status 8464 is logged. 
   2. TRDC1 was not selected for PAS_SYNC and it also has the old PAS, Active Directory replication status 0 (Successful) is logged. 
   3. ChildDC1 holds a writable copy of the CHILD partition so that it has all attributes for this partition. However, there is a pre-existing issue that causes Active Directory replication to fail with error 8606.
  
      :::image type="content" source="./media/replication-error-8464/status-8464-logged-for-dc2.png" alt-text="Screenshot 1":::

2. The destination domain controller selects a new source (TRDC1) for the PAS_SYNC task. 
   1. TRDC1 also has the old PAS so replication is delayed, and status 8464 is logged. 
   2. DC2 also has the old PAS. However, it is not selected for PAS_Sync on this interval, and replication is completed correctly. Therefore, status 0 is logged. 
   3. Active Directory replication still fails with ChildDC1 because of an unrelated lingering objects issue exists (abandoned objects).
  
        :::image type="content" source="./media/replication-error-8464/status-8464-logged-for-trdc1.png" alt-text="Screenshot 2":::

3. PAS_SYNC toggles back to the other outdated replica (DC2). 
   1. Meanwhile we correct the replication issue on ChildDC1. 
   2. Replication is delayed from DC2, and status 8464 is logged. 
   3. Replication proceeds successfully from TRDC1. 
   4. Replication proceeds successfully from ChildDC1 (but it is not selected for PAS_Sync on this cycle).

        :::image type="content" source="./media/replication-error-8464/correct-issues-on-childdc1.png" alt-text="Screenshot 3":::

4. A suitable domain controller is finally selected for PAS_SYNC (ChildDC1). 
   1. Replication proceeds as usual from DC2 and TRDC1 (these attempts are completed before PAS_Sync). 
   2. Replication proceeds as usual from ChildDC1and PAS_SYNC is complete.

        :::image type="content" source="./media/replication-error-8464/childdc1-selected-for-pas-sync.png" alt-text="Screenshot 4":::

5. The destination domain controller finally has the updated PAS (from the last interval). 
    1. Replication from DC2 and TRDC1 is now both delayed because the source domain controllers are outdated. The same Active Directory replication status is logged for this issue. 
    2. Replication is complete successfully from ChildDC1.

        :::image type="content" source="./media/replication-error-8464/status-8464-on-dc2-trdc1.png" alt-text="Screenshot 5":::

6. In between the previous replication interval and the next one, DC2's copy of the partial attribute set for the CHILD domain is also updated (not pictured though). 
   1. Because both the destination domain controller (DC1) and source domain controllers (DC2 and ChildDC1 *) have the updated PAS, replication is completed correctly.

    *ChildDC1 has a full set of attributes for the partition (not just the PAS). 
   1. Replication is delayed from TRDC1 because it still has the old PAS.

        :::image type="content" source="./media/replication-error-8464/replication-delayed-from-trdc1.png" alt-text="Screenshot 6":::

7. In between the previous replication interval and the next one, TRDC1's copy of the partial attribute set for the CHILD domain is also updated (not pictured though). 
   1. Replication is completed correctly from all partners, because the destination domain controller and sources all have the same attributes for the PAS.

        :::image type="content" source="./media/replication-error-8464/replication-completed-from-all-partners.png" alt-text="Screenshot 7":::
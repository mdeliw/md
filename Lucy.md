[toc]

# Postgres

- What is the topology for E1?
- What is the topology for E2?
- What is the topology for E3 - IP2 and IPC1?
- What is the proposed replication model in each environment?
- In E1, what is the backup proposal? 
  - What scenarios constitute a non-recoverable data loss? For example, you write to Master, replication has not happened, and the disk associated with master and the append-only log crashes etc.
- In a master / slave configuration, what are the possible scenarios where master is considered down and a slave has to become the master?
  - How can Ops detect that master is down?
  - What steps (time it can take) to switch a slave to master?
  - What happens when slave is lagging master and needs to become a master? Will we support this, or wait for Slave to catch up to latest replication logs and then become Master?
  - What steps (time it can take) to switch original master back to being a master?
  - How would we know that slave(s) are in sync or lagging master? 
    - master is accessible
    - master is not accessible
- What happens when E3 IPC2 is committed (one or both AZs), but before IPC1 is commited, the network goes down, and then IPC2 goes down.
- Can IPC1 be slave and also accept writes from sources other than replication logs?

# App - E1

- How does the App propose to manage commits and sync or async form of replication?
- How does App detect a need to failover? 
- How does the App propose to handle the failover?
  - Automated or manual etc?
  - What happens if the slave that becomes a master is lagging behing the master, and is missing commits?

# Planned release - E3 Soft Launch

- What does E3 mean?

  - Blue (Prod)
  - Green (Soft Launch)
  - IPC2 and IPC1
  - IPC2 or IPC1 -> later synched?
    - What happens if IPC1 is down, and IPC2 is up?
    - What happens if IPC2 is down and IPC1 is up?
    - What does successful release mean?

- How does AMP propose use of blue / green to handle live traffic?

  - IPC1 will also have blue / green?

- What is the proposal for state of rule when in E3? For example:

  - Use rule version and / or rule state?
  - Just released and ready for use in Soft Launch
  - Ready for use in Production
  - Moved from above states to inactive?
  - Moved from inactive to one of the above states?
  - Are there multiple forms of inactive? such as:
    - moved from SL to inactive and hence can only propogate up to SL, and not to Prod
    - moved from Prod to inactive and hence can directly propogate to Prod
    - any other forms? 

- Decouple rule distribution and App notification - that is *when to inform App* of a rule that was previously distributed.

  - Possibly another portal to orchestrate rules availability notification to pods

- Scenario: SL1

  - Lucy releases version 3. 
  - Prod is on version 1 and plans to stay on version 1 even if it is rebooted.
  - SL is testing version 2 and plans to continue to test version 2 even if SL is rebooted. 

  I would like SL to be influenced on when to switch over to version 3 regardless of the release strategy. 

- Scenario: SL2

  SL completed testing version 2 and now wishes to switch over to version 3, what next?

- Scenario: SL3

  SL was failed over to IPC1 and version 3 does not exist in IPC1, but version 2 does. What next?

- Scenario: SL3

  In SL testing of version 3, defects are found, what next?

- Scenario: SL4

  SL successfully tested version 3, what next? 

# Planned release - E3 Prod

- Rule state?
- Does App have the blue / green context?
- How do we envision notifying AMP of a availability of version of rule?
  - Control needed to decide which pods / environment needs to be notified.
  - Control needed to know if IPC1 is lagging IPC2.
- Scenario: PR1
  - Rule version 3 is ready for production. IPC2 is committed, IPC1 is not yet committed. 
  - IPC2 pods are notified or just the green becoming blue means the version 3 should be used?
  - IPC2 goes down, AMP fails over to IPC1, what next? 
    - Still running version 2?
    - IPC1 pods get notified to use version 3, but version 3 does not exist in IPC1.
- Scenario:
  - AMP running on IPC1
  - IPC2 pods comes online, but are not yet taking traffic. What rule version will the pods see?

# Hotfix - E3 Happy Path

This is the happy path of hotfix change made into IPC2 which then replicates into IPC1.

Assumptions

- hotfixes are tiny changes, low-risk changes, adhoc changes as a stop gap.
- the effort to automatically merge hotfix into main branch is likely complex - need to weigh benefits versus manually tracking the changes
- A workable solution is possible with systemic controls (that can be overridden)  coupled with manual controls to ensure changes are accurately put into production.

Proposal

- A separate authoring platform is necessary for hot fix for several reasons.

  - target is direct to E3 prod
  - needs to align to current AMP availability zone
  - what can be changed will be restricted
  - special handling required depending upon where the rules are getting updated (more later)
  - allow reads, but limit writes

- The risk of a planned change in pipeline potentially overriding a hot fix can be solved as follows:

  - At the time of initiating a release (e1 -> e2 -> e3), the platform should have a clear idea of what the previous snapshot of the planned e3 release was (this can be a checksum, remembered at each successful release). Say it is c1. 
  - During release of the current planned change, if the remembered / past checksum of e3 matches with what is in e3 - there were no interim hot fixes.
  - If however the checksums do not match, an interim hotfix has been made. Now we can continue to release the planned change regardless of whether it already has or does not have the hot fix. Post release, the state of the planned rule will be “not active / awaiting SL validation”- however we annotate the planned change with an indication that the change potentially overrides a hot fix change and needs a manual verification.
  - A separate portal platform will have access to all rules saved in SL and Prod, along with their states. When the planned change is to be activated, the portal can warn the admin of the risk of overriding a hot fix. 
  - The admin will have access to what changed in the hot fix, and  can verify if the planned change includes the hot fix or not - depending upon which the admin can make the appropriate decision. 
  - The portal will need to be able to highlight what change was introduced as hot fix. 
  - If above approach is acceptable, we can avoid implemeting sophisticated reverse cicd pipeline and selective merges.

  To solve

  - hot fix is released to IPC2, and has not been committed to IPC1. 
  - IPC2 goes down. AMP fails over, and the hot fix is not available in IPC1.
  - A reference to the hot fix is not available, so a fresh hot fix will need to be made and dispatched to IPC1.

# Hotfix - E3 Failover

IPC2 is down and AMP is on IPC1. The hotfix is authored and dispatched.

Assumption:

- AMP in IPC1 for extended periods of time is very unlikely. 
- AMP would be operating in a limited capacity - based on RDM availability.

Approach

- The E3 authoring platform will be aware that the destination is IPC1.
- As direct changes made to IPC1 can not be automatically replicated into IPC2, a separate approach is needed to move the changes to IPC2.
- The E3 authoring platform can support this function. The admin should have a view of which rule changes are in which E3 environment, and then should be able to take action to optionally manually move changes from IPC1 to IPC2.
  - Now will the one-way replication from IPC2 to IPC1 bring those changes back - need to discuss.

Scenario: IPC2 was down, and hotfix was authored and dispatched to IPC1.  Later IPC2 was brought online. 

- E3 authoring tool needs to reverse replicate the hotfix to IPC2.
- IPC2 pods need to started or notified

# Tooling

WIP
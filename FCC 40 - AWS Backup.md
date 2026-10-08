# FCC 40 - AWS Backup

> Video chapter [66:12:59](https://youtu.be/5JTBS09e7ew?t=238379) to 66:29:07 · 16 min total · 2 min lecture, 14 min lab
> Fast path: read this note and skip the video. The lab is a console click-through with long waits. Its one real lesson is that a backup plan does nothing until resources are assigned to it. Vault Lock, cross-account copies and restore testing are tested and not shown; they are in "Not in the video".

## Summary

- AWS Backup is one place to schedule, store and restore backups for many services: EC2, EBS, RDS, Aurora, DynamoDB, EFS, FSx, S3, Storage Gateway and more.
- A **backup plan** holds rules: when to back up, the backup window, the lifecycle (move to cold storage, delete) and where to copy.
- A **backup vault** stores the backups, which are called recovery points. It has its own KMS key.
- Resources join a plan by **tag** (preferred) or by resource ID. A plan with no resource assignment never runs.
- Backups can be copied to other Regions and other accounts, and managed for a whole organisation from one account.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [66:14:36](https://youtu.be/5JTBS09e7ew?t=238476) | 14m | On-demand backup of a DynamoDB table, delete the table, restore it under a new name, then a scheduled backup plan | The on-demand backup and restore work. The plan never fires, and after waiting an hour he finds the cause: no resources were assigned. After assigning the table, a job appears on the next run. The plan cannot be deleted until its resource assignment is removed. | Skip |

### Useful moments inside the lab

| Time | What you learn |
|------|----------------|
| [66:16:38](https://youtu.be/5JTBS09e7ew?t=238598) | On-demand backup form: retention, and "move to cold storage". Cold storage adds a 90-day minimum, so total retention must be at least the transition days plus 90. |
| [66:17:39](https://youtu.be/5JTBS09e7ew?t=238659) | Vault and IAM role are chosen per backup. The default role fails once with a permissions error and works on the second try. |
| [66:19:42](https://youtu.be/5JTBS09e7ew?t=238782) | The backup also shows under the table's Backups tab in the DynamoDB console |
| [66:19:55](https://youtu.be/5JTBS09e7ew?t=238795) | He deletes the table, then restores from the vault to a new table name |
| [66:21:43](https://youtu.be/5JTBS09e7ew?t=238903) | Restore progress is under Jobs, Restore jobs. The first status he saw was misleading. |
| [66:22:44](https://youtu.be/5JTBS09e7ew?t=238964) | Backup rule form: vault, frequency, start window, completion window, continuous backup, cold storage, retention, copy to another Region |
| [66:25:47](https://youtu.be/5JTBS09e7ew?t=239147) | The fix: "Assign resources" to the plan, by resource type and ID |
| [66:27:49](https://youtu.be/5JTBS09e7ew?t=239269) | Deleting a plan fails until its resource selections are deleted. Recovery points stay in the vault until they expire or you delete them. |

## Lecture index

| Time | Topic |
|------|-------|
| [66:12:59](https://youtu.be/5JTBS09e7ew?t=238379) | Supported services, backup plans, vaults, tags, cross-Region and cross-account, Backup Audit Manager |

## Notes

### Building blocks

| Piece | What it is |
|-------|------------|
| Backup plan | A policy made of one or more backup rules |
| Backup rule | Schedule, start window, completion window, lifecycle, target vault, optional copy to another Region or account |
| Resource assignment | Which resources the plan covers: by tag, by resource type or by ID, plus the IAM role AWS Backup uses |
| Backup vault | Container for recovery points, encrypted with a KMS key, with its own access policy |
| Recovery point | One backup of one resource |
| On-demand backup | A single backup made now, with no plan |
| Backup Audit Manager | Frameworks of controls that check your backup practice, and reports for auditors |

### Lifecycle and storage

| Setting | Detail |
|---------|--------|
| Warm storage | Where backups start; fast restore |
| Cold storage | Cheaper; a backup must stay there at least 90 days. Not every resource type supports it. |
| Retention | How long before the recovery point is deleted |
| Continuous backup | Point-in-time restore for supported services, up to 35 days back |
| Encryption | The vault's KMS key, which can differ from the key on the source resource |

### Not in the video

| Topic | Fact |
|-------|------|
| Backup Vault Lock | Makes a vault write once, read many. Governance mode can be removed by users with the right permission. Compliance mode cannot be removed by anyone once its cooling-off period ends. |
| Cross-account copies | Need AWS Organizations. The destination vault's access policy must allow the copy. |
| Organisation-wide plans | Backup policies in AWS Organizations push the same plan to every account |
| Restore testing | Runs test restores on a schedule to prove that backups can be recovered |
| Notifications | Job events go to EventBridge, or to an SNS topic set on the vault |
| Default role | `AWSBackupDefaultServiceRole`. A failed job is often a role without permission for that resource type. |
| Tags for assignment | The usual pattern: tag resources `backup=daily` and let the plan select on the tag, so new resources are covered automatically |

## Memory hooks

- Plan is the timetable, vault is the safe, recovery point is what is in the safe.
- A plan with no resources is a timetable with no trains.
- Cold storage: cheap, and you commit to 90 days.

## Exam traps

- "One central backup policy for many services and accounts": AWS Backup with Organizations backup policies.
- "Backups must not be deleted by anyone, including administrators, for seven years": Backup Vault Lock in compliance mode.
- "Keep a copy of backups in another Region or account for disaster recovery": a copy action in the backup rule.
- "Back up every resource tagged in a certain way, including future ones": assign resources to the plan by tag.
- "The backup plan exists but no backup jobs appear": no resource assignment, or the IAM role lacks permission.
- "Show auditors that backups meet policy": Backup Audit Manager.

## Outdated or wrong in the video

- **"Backup vaults are write once, read many" and "backups are immutable".** Only when Vault Lock is applied to the vault, or in a logically air-gapped vault. In a plain vault a user with permission can delete recovery points, as he does at the end of the lab. Very sure.
- **"Backups are incremental."** It depends on the resource type. EBS, RDS and EFS backups are incremental after the first one. DynamoDB backups are full copies. Fairly sure.
- **Logically air-gapped vault: "move things in afterwards, I assume an enterprise feature".** It is an ordinary vault type, and since late 2025 a plan can back up straight into it. Fairly sure.
- **"There is no way to queue a job from a plan."** True, but "Create on-demand backup" does the same job at once, and he had already used it. Very sure.

Related: [[FCC 05 - EBS]] · [[FCC 06 - RDS]] · [[FCC 07 - Amazon DynamoDB]] · [[FCC 11 - EFS]] · [[FCC 10 - Amazon S3 Glacier]] · [[FCC 00 - Watch guide]]

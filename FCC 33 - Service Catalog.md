# FCC 33 - Service Catalog

> Video chapter [63:40:12](https://youtu.be/5JTBS09e7ew?t=229212) to 63:47:40 · 7 min total · all lecture, no lab
> Fast path: read this note and skip the video. It is slides with console screenshots, and every point is below.

## Summary

- Service Catalog lets administrators publish approved CloudFormation templates as **products**. Users launch them from a catalogue instead of getting direct access to the underlying services.
- A **portfolio** holds three things: products, permissions (who may launch) and constraints (rules on how).
- Access is granted per portfolio to IAM groups, roles or users. Everyone with access sees every product in it.
- A **launch constraint** names an IAM role that does the provisioning, so the user needs no permissions on EC2, RDS and so on.
- A launched product is a **provisioned product**, which is a CloudFormation stack.

## Labs and demos

No lab or demo in this chapter.

## Lecture index

| Time | Topic |
|------|-------|
| [63:40:15](https://youtu.be/5JTBS09e7ew?t=229215) | What it is and its four advantages |
| [63:40:40](https://youtu.be/5JTBS09e7ew?t=229240) | Anatomy: administrator side and end-user side |
| [63:42:10](https://youtu.be/5JTBS09e7ew?t=229330) | The two types of user |
| [63:42:42](https://youtu.be/5JTBS09e7ew?t=229362) | Products |
| [63:43:35](https://youtu.be/5JTBS09e7ew?t=229415) | Portfolios and permissions |
| [63:44:30](https://youtu.be/5JTBS09e7ew?t=229470) | Constraints |
| [63:45:38](https://youtu.be/5JTBS09e7ew?t=229538) | End-user view: launching, versions, plans |
| [63:47:05](https://youtu.be/5JTBS09e7ew?t=229625) | Service actions |

## Notes

### Vocabulary

| Term | Meaning | What it is underneath |
|------|---------|-----------------------|
| Product | Defines the resources to launch; has versions | CloudFormation template |
| Portfolio | Container for products, permissions and constraints | |
| Constraint | A rule on how a product may be launched or used | |
| Provisioned product | A product someone has launched | CloudFormation stack |
| Provisioned product plan | Preview of what will be created or changed before launch | CloudFormation change set |
| Service action | A maintenance task users may run on their provisioned product | SSM document |

### Who does what

| User | Job |
|------|-----|
| Catalog administrator | Writes the templates, groups products into portfolios, sets constraints, manages the IAM roles, grants access |
| End user | Opens the catalogue in the console (or through the API), picks a product, names it, chooses a version, fills in the parameters, launches |

### The five constraint types

| Constraint | What it does |
|------------|--------------|
| Launch | Provisions with a named IAM role instead of the user's own credentials. Less permission for the user, more secure. |
| Notification | Sends the stack's events to an SNS topic |
| Template | Limits the parameter values a user may pick, for example only `t2.micro` |
| StackSet | Deploys the product across accounts and Regions with CloudFormation StackSets |
| Tag update | Allows or forbids users to change tags on a provisioned product |

### Other points

- Advantages he lists: standardisation, self-service discovery and launch, fine-grained access control, extensibility and version control.
- You cannot show some products to some users inside one portfolio. Make separate portfolios.
- A product is invisible until it is in a portfolio that has permissions attached.
- An AWS Budget can be attached to a product to track its cost.
- To delete a product, first remove it from every portfolio. He adds that no user may still have it provisioned.

### Not in the video

| Topic | Remember |
|-------|----------|
| TagOptions | A library of allowed tag keys and values the administrator attaches to portfolios or products. It enforces consistent tagging on whatever is launched. |
| Sharing | A portfolio can be shared with other accounts or with a whole AWS Organization |

## Memory hooks

- A vending machine: the administrator stocks it (portfolio), locks the options (constraints), the user presses a button (launch).
- Product is to provisioned product as template is to stack.
- CloudFormation says how to build it. Service Catalog says who may build which template, with which options.

## Exam traps

- "Let teams deploy approved, standard stacks themselves without broad IAM permissions": Service Catalog.
- "Users must launch a product without permissions on the services inside it": launch constraint with an IAM role.
- "Restrict which instance types users can choose": template constraint.
- "Different teams should see different products": separate portfolios.
- "Give users a restart button for what they launched": service action, backed by an SSM document.

## Outdated or wrong in the video

- **"Once a product is created it can only be edited, you can't delete it."** He contradicts this in his next sentence. A product can be deleted once it is removed from all portfolios. Very sure.
- **"Notification constraints send product notifications to a stack."** They send the stack's events to an SNS topic. Very sure.
- **"A product is just a CloudFormation template."** Since 2023 a product can also be a Terraform configuration. CloudFormation is still the answer the exam expects. Very sure.

Related: [[FCC 29 - AWS CloudFormation]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 22 - AWS Systems Manager]] · [[FCC 00 - Watch guide]]

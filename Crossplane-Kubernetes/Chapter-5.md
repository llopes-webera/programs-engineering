# Exploring Infrastructure Platform Patterns

## Evolving the APIs
Crossplane is primarily an API-based infrastructure automation platform. Changes to the APIs are inevitable as the business requirements and technology landscape evolve. We can classify these changes into three different buckets:
- API implementation change
- Non-breaking API contract change
- Breaking API contract change

### API Implementation Change
The compositions are mutable objects that can change forever, but individual `CompositionRevision` is immutable. `Composition` and `CompositionRevision` are in one-to-many relationships. We will have only one `CompositionRevision` active at any given instance.

**TIP**
*Each configuration state of the composition maps to a single CompositionRevision. Let’s say we are in revision 2 and changing the composition configuration the same as the first revision. A new revision is not created. Instead, revision 1 becomes active, making revision 2 inactive.*

The following diagram represents how Composition and CompositionRevision work together to evolve infrastructure API implementation continuously:
![Alt text](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781801811545/files/image/B17830_05_01.jpg)

Example of updating the composition patch with a `transform` function to multiply the disk size by four before patching.
```
patches:
    - type: FromCompositeFieldPath
      fromFieldPath: spec.parameters.size
      toFieldPath: spec.forProvider.settings.dataDiskSizeGb
      transforms:
      - type: math
        math:
          multiply: 4
```
After updating the composition, you will see two revisions. Only the latest revision will have the current flag of true.

## Unifying the automation
Managing external software resources with Crossplane is the crossroad for unifying infrastructure and application DevOps. We could package software and infrastructure dependencies into a single XRs. Such a complete package of applications and infrastructure introduces numerous advantages, some of which are listed here:

- The approach will unify the tooling and skills required for application and infrastructure automation.
- More importantly, the entire stack will enjoy the advantages of the Kubernetes operating model.
- Integrating vendor software into an enterprise ecosystem will become quicker and more standardized. Software vendors can quickly build packages that fit into different ecosystems. Currently, software vendors must custom-build for the individual cloud provider marketplace. This approach can assist in building a universal vendor software marketplace.
- We can easily apply the audit process to comply with any compliance standards. Previously, this would have been complicated as software and its infrastructure dependencies are spread about.

![Alt text](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781801811545/files/image/B17830_05_15.jpg)
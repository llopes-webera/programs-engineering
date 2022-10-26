# Composing Infrastructure with Crossplane

The ability to organize infrastructure recipes in a no-code way perfectly matches the organization's Agile expectation of building a lean platform team.
- **XRs** - CrossPlane Composite Resources.

(...) But they may not have experience in building APIs. Building an infrastructure platform with Crossplane will be a shift from these usual ways. Modern infrastructure platform developers should have both pieces of knowledge, that is, infrastructure and API engineering.

![Alt text](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781801811545/files/image/B17830_04_01.jpg)

Every element of the Crossplane composite is designed to cover infrastructure engineering practices from the perspective of an API.

### How do the XRs work?
The first purpose is to combine related **Managed Resources** (**MRs**) into a single stack and build reusable infrastructure template APIs.
The second one is to expose only limited attributes of the infrastructure API to the application team after abstracting all organization policies.
- Composite Resource Definition (XRD)
- Composition
- Claim
Relevant - **Metadata name**: it is a concatenation of the plural name and a dot followed by the API group.
	> Example: `<resource-plural-name>.<API-group>`

Once we are done with the API specification, the next step is to build the API implementation. Composition is the Crossplane construct used for providing API implementation.
The following figure represents the composition configuration options and the relationship between them:

![Alt text](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781801811545/files/image/B17830_04_03.jpg)

It provides polymorphic behavior for our infrastructure API to work based on the context. For example, we could have different compositions defined for production and staging.

![Alt text](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781801811545/files/image/B17830_04_04.jpg)

### Claim
XRs are cluster-level resources, while the claims are namespace level. It enables us to create namespace-level authorization. For example, we can assign different permissions for different product teams based on their namespace ownership.

![Alt text](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781801811545/files/image/B17830_04_05.jpg)

## Postprovisioning of an XR

Example of Propagating credentials back.
- Define the list of connection secret keys in the XRD using the ConnectionSecretKeys configuration.
- Configure the composing resources to define how to populate connection keys defined in the XRD. Connection details configuration can be of different types. The FromConnectionSecretKey type is correct when copying the secret from an existing secret key. We have the FromFieldPath type for copying the connection details from one of the composing resource fields.
- The claim or XR should save the Secrets using the WriteConnectionSecretToRef configuration.

![Alt text](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781801811545/files/image/B17830_04_06.jpg)

### Patch Status
`ToCompositeFieldPath` is a patch type for copying any attribute from a specific composed resource back into the XR.
```
- type: ToCompositeFieldPath
  fromFieldPath: status.atProvider.currentDiskSize
  toFieldPath: status.dbDiskSize
```

### Preprovisioned Resources
There are a few use cases where we may not create a new external resource and instead will reuse an existing provisioned resource. We will look at two such use cases in this section. The first use case is when we decide to cache the composed recourses because new resource provisioning may take too long to complete. The second use case is about importing the existing resources from the external provider into the Crossplane. The `crossplane.io/external-name` **annotation** can help with this.
```
annotations:
    # Annotation to provide existing resource named
    crossplane.io/external-name: alpha-beta-vpc
```

### Provisinoning Resources with a Claim
Note that the claims are namespace resources, and we provision them in the alpha namespace here.
```
apiVersion: alpha-beta.imarunrk.com/v1
kind: GCPdb
metadata:
  # Claims in alpha namespace
  namespace: alpha
  name: mysql-db
spec:
  # Refer to the mysql composition
  compositionRef:
    name: mysql
  # save connection details as secret - db-conn2
  writeConnectionSecretToRef:
    name: db-conn2
  parameters:
    size: SMALL
```

### Troubleshooting Tricks
When we start looking for issues, we take a top-down approach. This is because Crossplane follows the same convention as Kubernetes to hold the errors close to the resource where it happens.
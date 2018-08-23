---
title: Defining Rules
order: 305
---

# {{ page.title }}

This page describes how to define rules for Forseti Scanner.

---

## Defining custom rules

You can find some starter rules in the
[rules](https://github.com/GoogleCloudPlatform/forseti-security/tree/stable/rules)
directory. When you make changes to the rule files, upload them to your
Forseti bucket under `forseti-server-xxxx/rules/` or copy them to the `rules_path`
listed in `forseti_server_conf.yaml`.

## Cloud IAM policy rules

This section describes rules for Cloud Identity and Access Management (Cloud IAM).

### Rule definition

Forseti Scanner recognizes the following rule grammar in YAML or JSON:

```yaml
rules:
  - name: $rule_name
    mode: $rule_mode
    resource:
      - type: $resource_type
        applies_to: $applies_to
        resource_ids:
          - $resource_id1
          - $resource_id2
          - ...
    inherit_from_parents: $inherit_from
    bindings:
      - role: $role_name
        members:
          - $member1
          - $member2
          ...
```

* `name`
  * **Description**: The name of the rule.
  * **Valid values**: String.

* `mode`
  * **Description**: The mode of the rule.
  * **Valid values**: One of `whitelist`, `blacklist` or `required`.
  * **Note**:
    * `whitelist`: Allow the members defined.
    * `blacklist`: Block the members defined.
    * `required`: Defined members with the specified roles must be found in policy.

* `resource`
  * `type`
    * **Description**: The type of the resource.
    * **Valid values**: One of `organization`, `folder` or `project`.

  * `applies_to`
    * **Description**: What resources to apply the rule to.
    * **Valid values**: One of `self`, `children` or `self_and_children`.
    * **Note**:
      * `self`: Allow the members defined.
      * `children`: Block the members defined.
      * `self_and_children`: The rule applies to the specified resource and its child resources.

  * `resource_ids`
    * **Description**: A list of one or more resource ids to match.
    * **Valid values**: String, you can use `*` to match for all.

* `inherit_from_parents`
  * **Description**: A boolean that defines whether a specified resource inherits ancestor rules.
  * **Valid values**: One of `true` or `false`.

* `bindings`
  * **Description**: The
  [Policy Bindings](https://cloud.google.com/iam/reference/rest/v1/Policy#binding) to audit.
    * `role`
      * **Description**: A [Cloud IAM role](https://cloud.google.com/compute/docs/access/iam).
      * **Valid values**: String.
      * **Example values**: `roles/editor`, `roles/viewer`
    * `members`
      * **Description**: A list of Cloud IAM members. You can also use wildcards.
      * **Valid values**: String.
      * **Example values**: `serviceAccount:*@*gserviceaccount.com` (all service accounts) or
        `user:*@company.com` (anyone with an identity at company.com).


## Kubernetes Engine rules

### Rule definition

```yaml
rules:
  - name: Nodepool version not patched for critical security vulnerabilities
    resource:
      - type: organization
        resource_ids:
          - '*'
    check_serverconfig_valid_node_versions: false
    check_serverconfig_valid_master_versions: false
    allowed_nodepool_versions:
      - major: '1.6'
        minor: '13-gke.1'
        operator: '>='
      - major: '1.7'
        minor: '11-gke.1'
        operator: '>='
      - major: '1.8'
        minor: '4-gke.1'
        operator: '>='
      - major: '1.9'
        operator: '>='
 ```

* `name`
  * **Description**: The name of the rule.
  * **Valid values**: String.

* `resource`
  * `type`
    * **Description**: The type of the resource.
    * **Valid values**: One of `organization`, `folder` or `project`.

  * `resource_ids`
    * **Description**: A list of one or more resource ids to match.
    * **Valid values**: String, you can use `*` to match for all.

* `check_serverconfig_valid_node_versions`
  * **Description**: If true, will raise a violation for any node pool running a version
  that is not listed as supported for the zone the cluster is running in.
  * **Valid values**: One of `true` or `false`.

* `check_serverconfig_valid_master_versions`
  * **Description**: If true, will raise a violation for any cluster running an out of
  date master version. New clusters can only be created with a supported master version.
  * **Valid values**: One of `true` or `false`.

* `allowed_nodepool_versions`
  * **Description**: Optional, if not included all versions are allowed.
  The list of rules for what versions are allowed on nodes.
    * `major`
      * **Description**: The major version that is allowed.
      * **Valid values**: String.
      * **Example values**: `1.6`, `1.7`, `1.8`

    * `minor`
      * **Description**: Optional, the minor version that is allowed. If not included, all minor
      versions are allowed.
      * **Valid values**: String.
      * **Example values**: `11-gke.1`, `12-gke.1`

    * `operator`
      * **Description**: Optional, defaults to =, can be one of (=, >, <, >=, <=). The operator
      determines how the current version compares with the allowed version. If a minor version is
      not included, the operator applies to major version. Otherwise it applies to minor versions
      within a single major version.
      * **Valid values**: String.
      * **Example values**: `>=`

## Blacklist rules

### Rule definition

```yaml
rules:
  - blacklist: Emerging Threat blacklist
    url: https://rules.emergingthreats.net/fwrules/emerging-Block-IPs.txt
```

* **blacklist**: The name of your blacklist.
* **url**: URL that contains a list of IPs to check against.

## Google Group rules

### Rule definition

```yaml
- name: Allow my company users and gmail users to be in my company groups.
  group_email: my_customer
  mode: whitelist
  conditions:
    - member_email: '@MYDOMAIN.com'
    - member_email: '@gmail.com'
```

## Cloud Storage bucket ACL rules

### Rule definition

```yaml
rules:
  - name: sample bucket acls rule to search for public buckets
    bucket: '*'
    entity: AllUsers
    email: '*'
    domain: '*'
    role: '*'
    resource:
        - resource_ids:
          - YOUR_ORG_ID / YOUR_PROJECT_ID
```

* `name`
  * **Description**: The name of the rule.
  * **Valid values**: String.

* `resource`
  * `resource_ids`
    * **Description**: A list of one or more resource ids to match.
    * **Valid values**: String, you can use `*` to match for all.

* `bucket`
  * **Description**: The bucket name you want to audit.
  * **Valid values**: String, you can use `*` to match for all.

* `entity`
  * **Description**: The [ACL entity](https://cloud.google.com/storage/docs/access-control/lists) that holds the bucket permissions.
  * **Valid values**: String.
  * **Example values**: `AllUsers`

* `email`
  * **Description**: The email of the entity.
  * **Valid values**: String, you can use `*` to match for all.

* `domain`
  * **Description**: The domain of the entity.
  * **Valid values**: String, you can use `*` to match for all.

* `role`
  * **Description**: The access permission of the entity.
  * **Valid values**: String, you can use `*` to match for all.

For more information, refer to the
[BucketAccessControls](https://cloud.google.com/storage/docs/json_api/v1/objectAccessControls#resource)
documentation.

## Cloud Audit Logging rules

### Rule definition

```yaml
rules:
  - name: sample audit logging rule for data access logging
    resource:
      - type: project
        resource_ids:
          - '*'
    service: 'storage.googleapis.com'
    log_types:
      - 'DATA_READ'
      - 'DATA_WRITE'
    allowed_exemptions:
      - 'user:user1@MYDOMAIN.com'
      - 'user:user2@MYDOMAIN.com'
 ```

* `name`
  * **Description**: The name of the rule.
  * **Valid values**: String.

* `resource`
  * `type`
    * **Description**: The type of the resource.
    * **Valid values**: One of `organization`, `folder` or `project`.

  * `resource_ids`
    * **Description**: A list of one or more resource ids to match.
    * **Valid values**: String, you can use `*` to match for all.

* `service`
  * **Description**: The service on which logs must be enabled. The special value of `allServices` denotes audit logs for all services.
  * **Valid values**: String.
  * **Example values**: `allServices`, `storage.googleapis.com`

* `log_types`
  * **Description**: The required log types.
  * **Valid values**: One of `AUDIT_READ`, `DATA_READ` or `DATA_WRITE`.

* `allowed_exemptions`
  * **Description**: Optional, a list of allowed exemptions in the audit logs for this service.
  * **Valid values**: String.
  * **Example values**: `user:user1@MYDOMAIN.com`


## Cloud SQL rules

### Rule definition

```yaml
rules:
  - name: sample Cloud SQL rule to search for publicly exposed instances
    instance_name: '*'
    authorized_networks: '0.0.0.0/0'
    ssl_enabled: 'False'
    resource:
      - type: organization
        resource_ids:
          - YOUR_ORG_ID / YOUR_PROJECT_ID
 ```

* `name`
  * **Description**: The name of the rule.
  * **Valid values**: String.

* `resource`
  * `type`
    * **Description**: The type of the resource.
    * **Valid values**: One of `organization`, `folder` or `project`.

  * `resource_ids`
    * **Description**: A list of one or more resource ids to match.
    * **Valid values**: String, you can use `*` to match for all.

* `instance_name`
  * **Description**: The Cloud SQL instance to which you want to apply the rule.
  * **Valid values**: String, you can use `*` to match for all.

* `authorized_networks`
  * **Description**: The allowed network.
  * **Valid values**: String.
  * **Example values**: `0.0.0.0/0`

* `ssl_enabled`
  * **Description**: Whether SSL should be enabled.
  * **Valid values**: One of `true` or `false`.

## BigQuery rules

### Rule definition

BigQuery scanner rules serve as blacklists, for example:

```yaml
rules:
  - name: sample BigQuery rule to search for public datasets
    dataset_id: '*'
    special_group: 'allAuthenticatedUsers'
    user_email: '*'
    domain: '*'
    group_email: '*'
    role: '*'
    resource:
      - type: organization
        resource_ids:
          - YOUR_ORG_ID / YOUR_PROJECT_ID
```

* `name`
  * **Description**: The name of the rule.
  * **Valid values**: String.

* `resource`
  * `type`
    * **Description**: The type of the resource.
    * **Valid values**: One of `organization`, `folder` or `project`.

  * `resource_ids`
    * **Description**: A list of one or more resource ids to match.
    * **Valid values**: String, you can use `*` to match for all.

* `dataset_id`
  * **Description**: The BigQuery dataset to which you want to apply the rule.
  * **Valid values**: String, you can use `*` to match for all.

* `special_group`
  * **Description**: The special group.
  * **Valid values**: String, you can use `*` to match for all.

* `domain`
  * **Description**: Domain.
  * **Valid values**: String, you can use `*` to match for all.

* `role`
  * **Description**: The BigQuery dataset to which you want to apply the rule.
  * **Valid values**: String, you can use `*` to match for all.

* `group_email`
  * **Description**: Group email.
  * **Valid values**: String, you can use `*` to match for all.

* `role`
  * **Description**: Role.
  * **Valid values**: String, you can use `*` to match for all.

The BigQuery Scanner rules specify entities that aren't allowed to access
your datasets. When you set a value of `*` for `special_group`, `user_email`,
`domain`, and `group_email`, Scanner checks to make sure that no entities can
access your datasets. If you specify any other value, Scanner only checks to
make sure that the entity you specified doesn't have access.

## Enabled APIs rules

### Rule definition

```yaml
rules:
  - name: sample enabled APIs whitelist rule
    mode: whitelist
    resource:
      - type: project
        resource_ids:
          - '*'
    services:
      - 'bigquery-json.googleapis.com'
      - 'compute.googleapis.com'
      - 'logging.googleapis.com'
      - 'monitoring.googleapis.com'
      - 'pubsub.googleapis.com'
      - 'storage-api.googleapis.com'
      - 'storage-component.googleapis.com'
 ```

* `name`
  * **Description**: The name of the rule.
  * **Valid values**: String.

* `mode`
  * **Description**: The mode of the rule.
  * **Valid values**: One of `whitelist`, `blacklist` or `required`.
  * **Note**:
    * `whitelist`: Allow only the APIs listed in `services`.
    * `blacklist`: Block the APIs listed in `services`.
    * `required`: All APIs listed in `services` must be enabled.

* `resource`
  * `type`
    * **Description**: The type of the resource.
    * **Valid values**: One of `organization`, `folder` or `project`.

  * `applies_to`
    * **Description**: What resources to apply the rule to.
    * **Valid values**: One of `self`, `children` or `self_and_children`.
    * **Note**:
      * `self`: Allow the members defined.
      * `children`: Block the members defined.
      * `self_and_children`: The rule applies to the specified resource and its child resources.

  * `resource_ids`
    * **Description**: A list of one or more resource ids to match.
    * **Valid values**: String, you can use `*` to match for all.

* `services`
  * **Description**: The list of services to whitelist/blacklist/require.
  * **Valid values**: String.
  * **Example values**: `bigquery-json.googleapis.com`, `logging.googleapis.com`

## Forwarding rules

### Rule definition

```yaml
rules:
  - name: Rule Name Example
    target: Forwarding Rule Target Example
    mode: whitelist
    load_balancing_scheme: EXTERNAL
    ip_protocol: ESP
    ip_address: "198.51.100.46"
```

* `name`
  * **Description**: The name of the rule.
  * **Valid values**: String.

* `target`
  * **Description**: The URL of the target resource to receive the matched traffic.
  * **Valid values**: String.

* `mode`
  * **Description**: The mode of the rule.
  * **Valid values**: Current only support `whitelist` mode.
  * **Note**:
     * `whitelist`: Ensure each forwarding rule only directs to the intended target instance.

* `load_balancing_scheme`
  * **Description**: What the ForwardingRule will be used for.
  * **Valid values**: One of `INTERNAL` or `EXTERNAL`.

* `ip_protocol`
  * **Description**: The IP protocol to which this rule applies.
  * **Valid values**: One of `TCP`, `UDP`, `ESP`, `AH`, `SCTP`, or `ICMP`.

* `ip_address`
  * **Description**: The IP address for which this forwarding rule serves.
  * **Valid values**: String.
  * **Example values**: `198.51.100.46`

To learn more, see the
[ForwardingRules](https://cloud.google.com/compute/docs/reference/latest/forwardingRules)
documentation.

## Cloud IAP rules

This section describes rules for Cloud Identity-Aware Proxy (Cloud IAP).

### Rule definition

```yaml
rules:
  # custom rules
  - name: Allow direct access from debug IPs and internal monitoring hosts
    resource:
      - type: organization
        applies_to: self_and_children
        resource_ids:
          - YOUR_ORG_ID
    inherit_from_parents: true
    allowed_direct_access_sources: '10.*,monitoring-instance-tag'
```

* `name`
  * **Description**: The name of the rule.
  * **Valid values**: String.

* `mode`
  * **Description**: The mode of the rule.
  * **Valid values**: One of `whitelist`, `blacklist` or `required`.
  * **Note**:
    * `whitelist`: Allow the members defined.
    * `blacklist`: Block the members defined.
    * `required`: Defined members with the specified roles must be found in policy.

* `resource`
  * `type`
    * **Description**: The type of the resource.
    * **Valid values**: One of `organization`, `folder` or `project`.

  * `applies_to`
    * **Description**: What resources to apply the rule to.
    * **Valid values**: One of `self`, `children` or `self_and_children`.
    * **Note**:
      * `self`: Allow the members defined.
      * `children`: Block the members defined.
      * `self_and_children`: The rule applies to the specified resource and its child resources.

  * `resource_ids`
    * **Description**: A list of one or more resource ids to match.
    * **Valid values**: String, you can use `*` to match for all.

* `inherit_from_parents`
  * **Description**: A boolean that defines whether a specified resource inherits ancestor rules.
  * **Valid values**: One of `true` or `false`.

* `allowed_direct_access_sources`
  * **Description**:  Comma-separated list of globs that are matched against the IP ranges and tags in your
  firewall rules that allow access to services in your GCP environment.
  * **Valid values**: String.
  * **Example values**: `10.*,monitoring-instance-tag`

## Instance Network Interface rules

### Rule definition

```yaml
rules:
  # This rule helps with:
  # #1 Ensure instances with external IPs are only running
  # on whitelisted networks
  # #2 Ensure instances are only running on networks created in allowed
  # projects (using XPN)
  - name: all networks covered in whitelist
    project: '*'
    network: '*'
    is_external_network: True
    # this would be a custom list of your networks/projects.
    whitelist:
       project-1:
        - network-1
       project-2:
        - network-2
        - network-2-2
       project-3:
        - network-3
```

* `name`
  * **Description**: The name of the rule.
  * **Valid values**: String.

* `project`
  * **Description**: Project id.
  * **Valid values**: String, you can use `*` to match for all.

* `network`
  * **Description**: Network.
  * **Valid values**: String, you can use `*` to match for all.

* `whitelist`
  * **Description**: The whitelist describes which projects and networks for which VM
  instances can have external IPs.
  * **Valid values**: project/networks pairs.
  * **Example values**: The following values would specify that VM instances in
  project_01’s network_01 can have external IP addresses:

    ```
    project_01:
    - network_01
    ```

 ## Service Account Key rules

 ### Rule definitions

 ```yaml
 rules:
  # The max allowed age of user managed service account keys (in days)
  - name: Service account keys not rotated
    resource:
      - type: organization
        resource_ids:
          - '*'
    max_age: 100 # days
 ```

* `name`
  * **Description**: The name of the rule
  * **Valid values**: String.

* `type`
  * **Description**: The type of the resource this rule applies to.
  * **Valid values**: String, one of `organization`, `folder` or `project`.

* `resource_ids`
  * **Description**: The id of the resource this rule applies to.
  * **Valid values**: String, you can use `*` to match for all.

* `max_age`
  * **Description**: The maximum number of days at which your service account keys can exist before rotation is required.
  * **Valid values**: String, number of days.
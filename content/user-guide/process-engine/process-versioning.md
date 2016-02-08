---

title: 'Process Versioning'
weight: 110

menu:
  main:
    identifier: "user-guide-process-engine-versioning"
    parent: "user-guide-process-engine"

---


# Versioning of Process Definitions

Business Processes are by nature long running. The process instances will maybe last for weeks, or months. In the meantime the state of the process instance is stored to the database. But sooner or later you might want to change the process definition even if there are still running instances.

This is supported by the process engine:

* If you redeploy a changed process definition you get a new version in the database.
* Running process instances will continue to run in the version they were started in.
* New process instances will run in the new version - unless specified explicitly.
* Support for migrating process instances to new a version is supported within certain limits.

So you can see different versions in the process definition table and the process instances are linked to this:

{{< img src="../img/versioning.png" title="Versioning" >}}

{{< note title="Multi-Tenancy" class="info" >}}
If you are using [multi-tenancy with tenant identifiers]({{< relref "user-guide/process-engine/multi-tenancy.md#one-process-engine-with-tenant-identifiers" >}}) then each tenant has his own process definitions which have versions independent from other tenants. See the [multi-tenancy section]({{< relref "user-guide/process-engine/multi-tenancy.md#versioning" >}}) for details.
{{< /note >}}


# Which Version Will be Used

When you start a process instance

* by **key**: It starts an instance of the **latest deployed version** of the process definition with the key.
* by **id**: It starts an instance of the deployed process definition with the database id. By using this you can start a **specific version**.

The default and recommended usage is to just use `startProcessInstanceByKey` and always use the latest version:

```java
processEngine.getRuntimeService().startProcessInstanceByKey("invoice");
// will use the latest version (2 in our example)
```

If you want to specifically start an instance of an old process definition, use a Process Definition Query to find the correct ProcessDefinition id and use `startProcessInstanceById`:

```java
ProcessDefinition pd = processEngine.getRepositoryService().createProcessDefinitionQuery()
    .processDefinitionKey("invoice")
    .processDefinitionVersion(1).singleResult();
processEngine.getRuntimeService().startProcessInstanceById(pd.getId());
```

When you use [BPMN CallActivities]({{< relref "reference/bpmn20/subprocesses/call-activity.md" >}}) you can configure which version is used:

```xml
<callActivity id="callSubProcess" calledElement="checkCreditProcess"
  camunda:calledElementBinding="latest|deployment|version"
  camunda:calledElementVersion="17">
</callActivity>
```

The options are

* latest: use the latest version of the process definition (as with `startProcessInstanceByKey`).
* deployment: use the process definition in the version matching the version of the calling process. This works if they are deployed within one deployment - as then they are always versioned together (see [Process Application Deployment]({{< relref "user-guide/process-applications/the-processes-xml-deployment-descriptor.md#deployment-descriptor-process-application-deployment" >}}) for more details).
* version: specify the version hard coded in the XML.


# Key vs. ID of a Process Definition

You might have spotted that two different columns exist in the process definition table with different meanings:

* Key: The key is the unique identifier of the process definition in the XML, so its value is read from the id attribute in the XML:

    ```xml
    <bpmn2:process id="invoice" ...
    ```

* Id: The id is the database primary key and an artificial key normally combined out of the key, the version and a generated id (note that the ID may be shortened to fit into the database column, so there is no guarantee that the id is built this way).


# Migration of Process Instances

The engine provides a simple API to migrate running processes to a new process
definition. The API allows to specify a migration plan, which contains the
ids of the source and target process definition as well as a list of migration
instructions. A migration instruction specifies a mapping from activities from
the source process definition to activities in the target process definition.
Such a migration plan can be used to migrate instances from the source process
definition to the new one.

An example API to create a migration plan:

```Java
MigrationPlan migrationPlan = processEngine.getRuntimeService()
  .createMigrationPlan("testProcess:1", "testProcess:2")
  .mapActivities("subProcess", "subProcess")
  .mapActivities("userTask", "userTask")
  .build();
```

A migration plan will be validated after creation if it contains migration
instructions which are currently not supported by the engine.

{{< note title="Limitations" class="warning" >}}
Currently the migration API only supports a specific type of migration
instructions:

- a migration instruction can only have one source activity
  and one target activity
- migrated activities must have the same type, i.e. an user task can only
  be migrated to an user task
- only user tasks and sub processes can be migrated
- not supported are multi instance activities and activities with boundary
  events
{{< /note >}}


## Generating a migration plan

It is possible to generate a migration plan which calculates all migration
instructions for two process definitions which map equal activities. This way
only mappings of activities which changed have to be specified.

For example the following migration plan contains all mappings for equal
activities and an additional mapping for `oldTask` to `newTask`:

```Java
MigrationPlan migrationPlan = processEngine.getRuntimeService()
  .createMigrationPlan("testProcess:1", "testProcess:2")
  .mapEqualActivities()
  .mapActivities("oldTask", "newTask")
  .build();
```

The auto migration plan generation does not guarantee to create a migration
plan with all possible instructions. This limitation is introduced to ensure to
only generate valid instructions which can always be executed. In some cases it
is not possible to ensure that a instruction is valid before executing it.
These kind of instructions will not be added by the migration plan generator.

## Executing a migration plan

A migration plan can be executed for process instances of the source process
definition.

```Java
RuntimeService runtimeService = processEngine.getRuntimeService();

MigrationPlan migrationPlan = runtimeService
  .createMigrationPlan("testProcess:1", "testProcess:2")
  .mapEqualActivities()
  .mapActivities("oldTask", "newTask")
  .build();

List<ProcessInstance> processInstances = runtimeService
  .createProcessInstanceQuery()
  .processDefinitionId("testProcess:1")
  .listPage(0, 10);

runtimeService.executeMigrationPlan(migrationPlan, processInstances);
```

This will execute the migration plan for every process instance ensuring
that the migration plan contains a migration instruction for all active
activities of the process instance. Otherwise the migration plan is incomplete
and the migration fails.

## Migration Procedure

The migration of a process instances follows some simple rules:

1. The migration plan contains all executed instructions, i.e. there are no
   implicit mappings.
2. As much as possible of the old process instance is preserved.
3. Existing process scopes are not updated, i.e. ignoring new boundary events.
4. Removed process scopes are deleted with history updates, listener and
   input/output mapping invocation.
5. Added process scopes are created like they where executed by the process
   engine, i.e. history entires, listener and input/output mapping invocation.

The migration procedure is compound of the following phases:

1. Calculated the activities to migrate for the process instance.
2. Remove scopes which no longer exist in the new process definition beginning
   from the top most scope. This ensures that scopes are removed in the reverse
   order they were created like during the normal process flow.
3. Add scopes for the new process definition beginning from the process
   definition scope. This ensures that scopes are created in the same
   order as by the normal process flow.

Currently the migration is limited to **vertical** migration. Which means it
can remove and add scopes in the scope hierarchy of one activity. For example
an activity like an user task has a hierarchy of ancestor scopes, i.e. parent
sub processes. The migration allows to remove or add scopes to this hierarchy.

**Horizontal** migration is not supported at the moment. Whereas horizontal
means to migrate an activity into a sibling scope, i.e. migrate a user task
from one sub process to a concurrent sub process.

---

title: "Update a Glassfish Installation from 7.3 to 7.4"

menu:
  main:
    name: "Glassfish"
    identifier: "migration-guide-73-glassfish"
    parent: "migration-guide-73"

---

The following steps describe how to upgrade the Camunda artifacts on a Glassfish 3.1 application server in a shared process engine setting. For the entire procedure, refer to the [upgrade guide][upgrade-guide]. If not already done, make sure to download the [Camunda BPM 7.4 Glassfish distribution](https://app.camunda.com/nexus/content/groups/public/org/camunda/bpm/glassfish/camunda-bpm-glassfish/).

The upgrade procedure takes the following steps:

1. Uninstall the Camunda Applications and Archives
2. Replace Camunda Core Libraries
3. Replace Optional Camunda Libraries
4. Maintain the BPM Platform Configuration
5. Configure Process Engines
6. Install the Camunda Archive
7. Install the Web Applications

In each of the following steps, the identifiers `$*_VERSION` refer to the current version and the new versions of the artifacts.

# 1. Uninstall the Camunda Applications and Archives

First, uninstall the Camunda web applications, namely the Camunda REST API (artifact name like `camunda-engine-rest`) and the Camunda applications Cockpit, Tasklist and Admin (artifact name like `camunda-webapp`).

Uninstall the Camunda EAR. Its name should be `camunda-glassfish-ear-$PLATFORM_VERSION.ear`. Then, uninstall the Camunda job executor adapter, called `camunda-jobexecutor-rar-$PLATFORM_VERSION.rar`.

# 2. Replace Camunda Core Libraries

After shutting down the server, replace the following libraries in `$GLASSFISH_HOME/glassfish/lib` with their equivalents from `$GLASSFISH_DISTRIBUTION/modules/lib`:

* `camunda-engine-$PLATFORM_VERSION.jar`
* `camunda-bpmn-model-$PLATFORM_VERSION.jar`
* `camunda-cmmn-model-$PLATFORM_VERSION.jar`
* `camunda-xml-model-$PLATFORM_VERSION.jar`

Add or replace (if already present) the following libraries:

* `camunda-engine-dmn-$PLATFORM_VERSION.jar`
* `camunda-engine-feel-api-$PLATFORM_VERSION.jar`
* `camunda-engine-feel-juel-$PLATFORM_VERSION.jar`
* `camunda-dmn-model-$PLATFORM_VERSION.jar`
* `camunda-commons-logging-$COMMONS_VERSION.jar`
* `camunda-commons-typed-values-$COMMONS_VERSION.jar`
* `camunda-commons-utils-$COMMONS_VERSION.jar`

Add or replace (if already present) the following SLF4J libraries unless you use SLF4J anyway. In that case, make sure that this version is compatible with Camunda's version (TODO: welche Version setzen wir minimal voraus? Auf die Logging-Sektion linken, die Daniel noch schreiben will?)

* `slf4j-api-$SLF4J_VERSION.jar`
* `slf4j-jdk14-$SLF4J_VERSION.jar`

# 3. Replace Optional Camunda Dependencies

In addition to the core libraries, there may be optional artifacts in `$GLASSFISH_HOME/glassfish/lib` for LDAP integration, Camunda Spin, and Groovy scripting. If you use any of these extensions, the following upgrade steps apply:

## LDAP integration

Copy the following libraries from `$GLASSFISH_DISTRIBUTION/modules/lib` to the folder `$GLASSFISH_HOME/glassfish/lib` if present:

* `camunda-identity-ldap-$PLATFORM_VERSION.jar`

## Camunda Spin

Copy the following libraries from `$GLASSFISH_DISTRIBUTION/modules/lib` to the folder `$GLASSFISH_HOME/glassfish/lib` if present:

* `camunda-spin-core-$SPIN_VERSION.jar`

## Groovy Scripting

Copy the following libraries from `$GLASSFISH_DISTRIBUTION/modules/lib` to the folder `$GLASSFISH_HOME/glassfish/lib` if present:

* `groovy-all-$GROOVY_VERSION.jar`

# 4. Maintain the BPM Platform Configuration

If you have previously replaced the default BPM platform configuration by a custom configuration following any of the ways outlined in the [deployment descriptor reference][configuration-location], it may be necessary to restore this configuration. This can be done by repeating the configuration replacement steps for the upgraded platform.

# 5. Configure Process Engines

This section describes changes in the engine’s default behavior. While the change is reasonable, your implementation may rely on the previous default behavior. Thus, the previous behavior can be restored for shared process engines by explicitly setting a configuration option.

## Task Query Expressions

As of 7.4, the default handling of expressions submitted as parameters of task queries has changed. Passing EL expressions in a task query enables execution of arbitrary code when the query is evaluated. The process engine no longer evaluates these expressions by default and throws an exception instead. This behavior can be toggled in the process engine configuration using the properties `enableExpressionsInAdhocQueries` (default `false`) and `enableExpressionsInStoredQueries` (default `true`). To restore the engine's previous behavior, set both flags to `true`. See the user guide on [security considerations for custom code]({{< relref "user-guide/process-engine/securing-custom-code.md" >}}) for details.
This is already the default for Camunda BPM versions after and including 7.3.3 and 7.2.8.

## User Operation Log

The behavior of the [user operation log]({{< relref "user-guide/process-engine/history.md#user-operation-log" >}}) has changed, so that operations are only logged if they are performed in the context of a logged in user. This behavior can be toggled in the process engine configuration using the property `restrictUserOperationLogToAuthenticatedUsers` (default `true`). To restore the engine's prior behavior, i.e. to write log entries regardless of user context, set the flag to `false`.

Furthermore, with 7.4 task events are only logged when they occur in the context of a logged in user. Task events are accessible via the deprecated API `TaskService#getTaskEvents`. If you rely on this API method, the previous behavior can be restored by setting the flag `restrictUserOperationLogToAuthenticatedUsers` to `false`.

# 6. Install the Camunda Archive

First, install the camunda job executor resource adapter, namely the file `$GLASSFISH_DISTRIBUTION/modules/camunda-jobexecutor-rar-$PLATFORM_VERSION.rar`. Then, install the camunda EAR, i.e., the file `$GLASSFISH_DISTRIBUTION/modules/camunda-glassfish-ear-$PLATFORM_VERSION.ear`.

# 7. Install the Web Applications

## REST API

The following steps are required to upgrade the camunda REST API on a Glassfish instance:

1. Download the REST API web application archive from our [Maven Nexus Server](https://app.camunda.com/nexus/content/groups/public/org/camunda/bpm/camunda-engine-rest/). Or switch to the private repository for the enterprise version (User and password from license required). Choose the correct version named `$PLATFORM_VERSION/camunda-engine-rest-$PLATFORM_VERSION.war`.
2. Deploy the web application archive to your Glassfish instance.

## Cockpit, Tasklist, and Admin

The following steps are required to upgrade the camunda web applications Cockpit, Tasklist, and Admin on a Glassfish instance:

1. Download the camunda web application archive from our [Maven Nexus Server](https://app.camunda.com/nexus/content/groups/public/org/camunda/bpm/webapp/camunda-webapp-glassfish/). Or switch to the private repository for the enterprise version (User and password from license required). Choose the correct version named `$PLATFORM_VERSION/camunda-webapp-glassfish-$PLATFORM_VERSION.war`.
2. Deploy the web application archive to your Glassfish instance.

{{< note title="LDAP Entity Caching" class="info" >}}
It is possible to enable entity caching for Hypertext Application Language (HAL) requests that the camunda web applications make. This can be especially useful when you use camunda in combination with LDAP. To activate caching, the camunda webapp artifact has to be modified and the pre-built application cannot be used as is. See the [REST Api Documentation]({{< relref "reference/rest/overview/hal.md" >}}) for details.
{{< /note >}}

[configuration-location]: {{< relref "reference/deployment-descriptors/descriptors/bpm-platform-xml.md" >}}
[upgrade-guide]: {{< relref "update/minor/73-to-74/index.md" >}}

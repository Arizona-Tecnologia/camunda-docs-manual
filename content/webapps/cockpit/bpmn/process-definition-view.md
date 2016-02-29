---

title: 'Process Definition View'
weight: 20

menu:
  main:
    identifier: "user-guide-cockpit-pd-view"
    parent: "user-guide-cockpit-bpmn"
    pre: "Gain an aggregated overview over all instances of a given process definition and perform bulk operations."

---

{{< img src="../../img/cockpit-process-definitions-view.png" title="Process Definition View" >}}

The process definition view provides you with information about the definition and the status of a process. On the left side you can easily survey the versions of the process and how many instances of the versions are running. Incidents of all running process instances are displayed together with an instance counter label in the corresponding rendered diagram. So it is easy to locate [failed activities][failed-activities] in the process. Use the mouse to navigate through the diagram. By turning the mouse wheel you can zoom in out. Hold the left mouse button to pan the diagram in the desired direction.
In the `Process Instances` tab all running instances are listed in a tabular view. Besides information about start time, business key and state you can select an instance by ID and go down to the [process instance view][process-instance-view].

The `Called Process Definitions` tab displays the called child processes. In the column *Called Process Definition* the names of the called sub processes are listed. Click on the name to display the process in the process definition view. Please note that a filter called Parent is automatically set for the process so that you only see the instances that belong to the parent process. In the *Activity* column you can select the instance that is calling the child process.

The `Job Definitions` tab displays the job definitions that are linked to this process definition. Observe the name of the activity, the type of job, the configuration and the state thereof. You can also suspend and re-activate the job definition (see [Job Definition Suspension][job-definition-suspension] for more information). It is also possible to [set the priority of jobs]({{< relref "webapps/cockpit/bpmn/process-definition-view.md#set-job-priority">}}).


[failed-activities]: {{< relref "webapps/cockpit/bpmn/failed-jobs.md" >}}
[process-instance-view]: {{< relref "webapps/cockpit/bpmn/process-instance-view.md" >}}
[job-definition-suspension]: {{< relref "webapps/cockpit/bpmn/suspension.md#job-definition-suspension" >}}


# Filter

The filter function on the left side of the Process Definition View allows you to find certain instances by filtering for variables, business keys, start time and date, end time and date (enterprise edition) or by selecting the version of a process. Beyond that you can combine different filters as logical *AND* relation. Filter expressions on variables must be specified as `variableName OPERATOR value` where the *operator* may be one of the following terms `=`, `!=`, `>`, `>=`, `<`, `<=`, `like`. Apart from the `like` operator, the operator expressions do not have to be separated by spaces.
The `like` operator is for string variables only. You can use `%` as wildcard in the *value* expression. String and date values must be properly enclosed in `" "`.

{{< note title="Complexe Datatypes" class="info" >}}
  Please be aware that this feature does not support complex data types.
{{< /note >}}


**Filtering for process instances**

{{< img src="../../img/filter-examples/add-filter.png" title="Filter Example" >}}

Add a filter to the process definition view. Select a filter for variables, the start date and time or the business key of process instances.


**Business Key**

{{< img src="../../img/filter-examples/business-key.png" title="Filter Example" >}}

Add a filter for process instances by business key.


**Start Date and Time**

{{< img src="../../img/filter-examples/start-date.png" title="Filter Example" >}}

Add a filter for process instances by start date. Please note that the date must be set in accordance to the [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) standard.


**String variable**

{{< img src="../../img/filter-examples/var-string.png" title="Filter Example" >}}

Add a filter for process instances by filtering for a 'string' value. Please note that you need to encase the value in quotation marks.


**Boolean variable**

{{< img src="../../img/filter-examples/var-boolean.png" title="Filter Example" >}}

Add a filter for process instances by filtering for a 'boolean' value.


**Date variable**

{{< img src="../../img/filter-examples/var-date.png" title="Filter Example" >}}

Add a filter for process instances by filtering for a 'date' value. Please note that the date value must be set in accordance to the [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) standard and that you need to encase the value in quotation marks.


**Numeric variable**

{{< img src="../../img/filter-examples/var-numeric.png" title="Filter Example" >}}

Add a filter for process instances by filtering for a 'numeric' (double, integer, long or short) value.


{{< enterprise >}}
Please note that this feature is only included in the enterprise edition of the Camunda BPM platform, it is not available in the community edition.
{{< /enterprise >}}

**Filtering for completed and running process instances**

{{< img src="../../img/filter-examples/history-filter.png" title="Filter Example" >}}

Add a filter to the [process definition history view][process-definition-history-view]. Select filters for variables, the start date and time, the end date and time or the business key of process instances.


**End Date and Time**

{{< img src="../../img/filter-examples/end-date.png" title="Filter Example" >}}

Add a filter for process instances by end date. Please note that the date must be set in accordance to the [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) standard and that this option is only available in the [process definition history view][process-definition-history-view].


[process-definition-history-view]: {{< relref "webapps/cockpit/bpmn/process-history-views.md#process-definition-history-view" >}}


# Cancel Multiple Process Instances

{{< enterprise >}}
Please note that this feature is only included in the enterprise edition of the Camunda BPM platform, it is not available in the community edition.
{{< /enterprise >}}

{{< img src="../../img/cockpit-bulk-cancel.png" title="Bulk Cancel" >}}

Cancel multiple process instances at once by using this feature. In the process definition view, hit the {{< glyphicon name="remove-circle" >}} button on the right side. This opens a confirmation screen in which you can select which process instances to cancel. After you have selected which instances to cancel and confirmed the cancellation, the runtime data of the canceled instances will be deleted. Please note that only process instances in the current view can be canceled, i.e., a maximum of 50 process instances at once.


# Set Job priority

{{< img src="../../img/cockpit-set-job-priority.png" title="Set Job Priority" >}}

You can change the job priority by overriding the priority specified in the BPMN 2.0 XML. To do so, click on the {{< glyphicon name="cog">}} icon in the `Job Definitions` tab. In the opened dialog you can override the job priority. If an override priority is already set, you can clear it to use the priority specified in the XML again. It is also possible to include existing jobs when changing the priority.

By using the {{< glyphicon name="cog">}} button to the right of the process diagram, you can set the job priority of all jobs contained in the process definition at once.

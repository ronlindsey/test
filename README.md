# Dynamic Forms

Dynamic Forms is a collection of AngularJS components, an associated Web API plugin and table-driven configuration that renders html and allows for data to be retrieved, inserted, updated or deleted from an existing table.

- [Database Tables](#database-tables)
- [Record Mode](#record-mode)
	- [Multiple Record Mode](#multiple-record-mode)
	- [Single Record Mode](#single-record-mode)
- [Form Layout](#form-layout)
- [Child Dynamic Forms](#child-dynamic-forms)
- [Usage](#usage)
	- [Web Plugin Usage](#web-plugin-usage)
	- [Directive Only Usage](#directive-only-usage)

## Database Tables
Dynamic Forms uses a number of tables to configure the system. All tables belong to the "dyn" schema.
- [DynamicFormAncestor](#dynamicformancestor)
- [DynamicForm](#dynamicform)
- [DynamicFormField](#dynamicformfield)
- [DynamicFormControlType](#dynamicformfieldcontroltype)
- [DynamicFormList](#dynamicformlist)
- [DynamicFormOption](#dynamicformoption)
- [DynamicFormRecordMode](#dynamicformrecordmode)
- [DynamicFormCustomHtml](#dynamicformcustomhtml)
- [DynamicFormEvent](#dynamicformevent)
- [DynamicFormFieldToEvent](#dynamicformfieldtoevent)

#### DynamicFormAncestor
This table contains the entity that is the highest in the chain of parent/child relationships. Optional.

Fields:
- DynamicFormAncestorId (int) - primary key for the table
- SourceTableName (varchar(50))- the name of the table
- SourceTablePrimaryKey (varchar(50)) - the primary key field name of the source table
- InsertedDate (datetime) - date the record was inserted
- UpdatedDate (datetime) - date the record was last updated
- UpdatedBy (varchar(256)) - the user that inserted or last updated the record
- ts (timestamp) - used for concurrency checks
- IsSetForDelete (bit) - flag for indirect deletes

#### DynamicForm
This table configures the source table for the data and form element layout.

Fields:
- DynamicFormId (int) - primary key for the table
- ParentId (int) - identifies the parent dynamic form record
- DynamicFormAncestorId (int) - foreign key to the [DynamicFormAncestor](#dynamicformancestor) table
- DynamicFormRecordModeId (int) - foreign key to the [DynamicFormRecordMode](#dynamicformrecordmode) table
- ParentKeyField (varchar(50)) - field name for the parent table lookup value (e.g. "AddressType" for the AltAddress.AddressType field)
- ParentKeyValue (varchar(25)) - value to match on the ParentKeyField field
- Name (varchar(50)) - name of the dynamic form
- DisplayName(varchar(50)) - name to display in the container heading
- SourceTable (varchar(50))- the name of the source table. This is the table from which data will be operated upon.
- SourceTableSchema (varchar(20)) - the name of the schema for the source table
- SourceTablePrimaryKey (varchar(50)) - the primary key field name of the source table
- SourceTableFilterField (varchar(50)) - name of the field for filtering of data (e.g. [Tracking_Number], [PaymentDetailID], [BorrowerID], etc.)
- SourceTableConcurrencyCheckField (varchar(50)) - name of the field used for concurrency check comparison. A concurrency error will be raised if the updated value for this field is not the same as the current value.
- SourceTableConcurrencyCheckUpdatedByField (varchar(50)) - name of the field identifiying the user during concurrency check comparison. If a concurrency error is raised, this field is used to determine the user name of the last user to update the record.
- DefaultSortField (varchar(50)) - name of the field to apply default sorting during grid rendering
- AllowAdd (bit) - default is true. Determines whether additions are allowed to the source table and whether the "Create New [DisplayName]" button is rendered.
- AllowDelete (bit) - default is true. Determines whether deletions are allowed from the source table and whether the "Delete" button is rendered on the form container or in the data grid (Multiple Record Mode only).
- AllowExport (bit) - default is true. Determines whether exporting of data is allowed and whether the "Export" button is rendered.

The following four fields serve as a replacement for the "ProtectedAttribute" on the API endpoint since all data, regardless of entity source, will flow through the same endpoints.

- ViewRight (varchar(20)) - fundamental right short name(s) that identifies whether the user has view rights to the source table. 
- EditRight (varchar(20)) - fundamental right short name(s) that identifies whether the user has the rights to the edit data.
- AddRight (varchar(20)) - fundamental right short name(s) that identifies whether the user has rights to add records to the source table. The "Create New [DisplayName]" button will be disabled/enabled (if AllowAdd is true) if the user is assigned this right.
- DeleteRight (varchar(20)) - fundamental right short name that identifies whether the user has the right to delete from the source table. The "Delete" button will be disabled/enabled (if AllowDelete is true) if the user is assigned this right.
- ExportRight (varchar(20)) - fundamental right short name(s) that identifies whether the user has the right to export data. The "Export" button will be disabled/enabled (if AllowExport is true) if the user is assigned this right.
- AddProcessingNoteOnDelete (bit) - identifies whether to add a record to the "Notes" table when deleting a record from the source table (note: should be extended to allow insert to a different table/field(s))
- UseCollapsibleContainer (bit) - default is true. Indicates whether the rendered container should be a collapsible container. Set this value to false to remove expand/collapse functionality. The container will be rendered as a panel with no heading.
- IsContainerInitiallyCollapsed (bit) - default is false. Indicates whether the rendered container should be initially collapsed (requires UseCollapsibleContainer to be set to 1).
- FetchDataOnExpand (bit) - default is false. Indicates whether the data should be lazy loaded when the user clicks on an initially collapsed container to expand it. Requires UseCollapsibleContainer and IsContainerIntiallyCollapsed to be set to 1. If either is set to false, this setting is ignored.
- InsertedDate (datetime) - date the record was inserted
- UpdatedDate (datetime) - date the record was last updated
- UpdatedBy (varchar(256)) - the user that inserted or last updated the record
- ts (timestamp) - used for concurrency checks
- IsSetForDelete (bit) - flag for indirect deletes

#### DynamicFormField
This table configures the fields for the associated source table.

Fields:
- DynamicFormFieldId (int) - primary key for the table
- DynamicFormId (int) - foreign key to the [DynamicForm](#dynamicForm) table
- DynamicFormFieldControlTypeId (int) - foreign key to the  [DynamicFormControlType](#dynamicFormControlType) table
- DynamicFormListId (int) - foreign key to the [DynamicFormList] table
- FieldName (varchar(50)) - name of the source table field
- DisplayName (varchar(50)) - value to display for label and/or grid header
- IsRequired (bit) - indicates whether the field is required for saving. If true, a red asterisk will be placed after the label, giving the user a visual cue. The engine will create the necessary html element that will display the appropriate error message to the user that this is a required field.
- DataGridColumnIndex (int) - determines the display order of the column in the data grid
- FormDisplayColumn (int) - indicates the start index for rendering the html element and associated label
- FormDisplayRow (int) - indicates the row for rendering the element and associated label
- FormDisplayColumnSpan (int) - indicates how many columns the element should span
- ShowInDataGrid (bit) - indicates whether a column will be rendered in the data grid for the field
- ShowOnForm (bit) - indicates whether to render the element and label on the form

The following three fields serve as the equivalent of applying "Entity Property Rights" logic. 

- RightsToView (varchar(50)) - fundamental right short name(s) that controls whether the data will be passed to the client. If the user is not assigned this right, the html element will be blank and read only, unless the user is assigned the right for [RightsToEdit] as described below.
- RightsToEdit (varchar(50)) - fundamental right short names(s) that controls whether the user has the rights to edit the property for an existing record. If the user is not assigned this right, the html element will not display the data (assuming he/she does not have the [RightToView] right), and set the readonly/disabled attribute so that the user cannot edit the data. If the user is assigned this right, the element will display the data and allow the user to edit the data.
- RightsToEditDuringRecordCreation (varchar(50)) - fundamental right short names(s) that controls whether the user has the rights to edit the field for a new record. If the user does not have the appropriate right, the element will be rendered with the "readonly" attribute.
- ValidationPattern (varchar(500)) - a javascript regex pattern used for input validation. The engine will create the necessary html element that will display the appropriate error message to the user that input is invalid.
- InputCssClass (varchar(100)) - additional css classes to render to the associated input element
- LabelCssClass (varchar(100)) - additional css classes to render to the associated label element
- GridColumnCssClass (varchar(100)) - css classes to apply to the data grid column
- AdditionalAttributes (varchar(250)) - additional attributes to apply to the rendered element
- BindSelectToValue (bit) - indicates whether the drop down should bind to the value attribute rather than the text attribute (applies only to select control type)
- IsViewSourcedNonUpdateable (bit) - indicates that the field is part of a view and should not be applied to the save operation. The save operation will ignore this field.
- InsertedDate (datetime) - date the record was inserted
- UpdatedDate (datetime) - date the record was last updated
- UpdatedBy (varchar(256)) - the user that inserted or last updated the record
- ts (timestamp) - used for concurrency checks
- IsSetForDelete (bit) - flag for indirect deletes

#### DynamicFormFieldControlType
This table configures the rendered element for the associated field.

Fields:
- DynamicFormControlTypeId (int) - primary key for the table
- Name (varchar(50)) - identifies the name of the control type and determines the html element(s) to render (e.g. text, select, textarea, etc.). These values do **_NOT_** directly correlate to an html element type.
- InsertedDate (datetime) - date the record was inserted
- UpdatedDate (datetime) - date the record was last updated
- UpdatedBy (varchar(256)) - the user that inserted or last updated the record
- ts (timestamp) - used for concurrency checks
- IsSetForDelete (bit) - flag for indirect deletes

#### DynamicFormList
This table configures a single list for associated select element options.

Fields:
- DynamicFormListId (int) - primary key for the table
- Name (varchar(50)) - identifies the name of the list
- InsertedDate (datetime) - date the record was inserted
- UpdatedDate (datetime) - date the record was last updated
- UpdatedBy (varchar(256)) - the user that inserted or last updated the record
- ts (timestamp) - used for concurrency checks
- IsSetForDelete (bit) - flag for indirect deletes

#### DynamicFormOption
This table configures a single list for associated select element options.

Fields:
- DynamicFormOptionId (int) - primary key for the table
- DynamicFormListId (int) - foreign key to the [DynamicFormList](#dynamicFormList) table
- Value (varchar(250)) - identifies the value used for the option value attribute
- Text (varchar(250)) - identifies the value used for the option text attribute
- InsertedDate (datetime) - date the record was inserted
- UpdatedDate (datetime) - date the record was last updated
- UpdatedBy (varchar(256)) - the user that inserted or last updated the record
- ts (timestamp) - used for concurrency checks
- IsSetForDelete (bit) - flag for indirect deletes

#### DynamicFormRecordMode
This table configures whether the rendered form will allow a single or multiple records.

Fields:
- DynamicFormOptionId (int) - primary key for the table
- Name (varchar(10)) - name of the mode ('Single', 'Multiple')
- InsertedDate (datetime) - date the record was inserted
- UpdatedDate (datetime) - date the record was last updated
- UpdatedBy (varchar(256)) - the user that inserted or last updated the record
- ts (timestamp) - used for concurrency checks
- IsSetForDelete (bit) - flag for indirect deletes

#### DynamicFormCustomHtml
This table configures custom html to be rendered. Custom html is rendered relative to the associated field.

Fields:
- DynamicFormCustomHtmlId (int) - primary key for the table
- DynamicFormFieldId (int) - foreign key to the [DynamicFormField](#dynamicFormField) table
- Offset (int) - determines whether to render the custom html before or after the associated element (1 for after, -1 for before)
- ColumnStart (int) - determines where to start rendering the custom html
- ColumnSpan (int) - determines how many columns the custom html will span
- IsNewRow (bit) - indicates whether the custom html should be rendered as a new row. If true, the html will be rendered in a new row rather than the same cell as the associated field.
- Html (varcahr(max)) - the custom html to render
- InsertedDate (datetime) - date the record was inserted
- UpdatedDate (datetime) - date the record was last updated
- UpdatedBy (varchar(256)) - the user that inserted or last updated the record
- ts (timestamp) - used for concurrency checks
- IsSetForDelete (bit) - flag for indirect deletes

#### DynamicFormEvent
This table contains AngularJS events.

Fields:
- DynamicFormEventId (int) - primary key for the table
- EventName (varchar(50)) - name of the angular event
- InsertedDate (datetime) - date the record was inserted
- UpdatedDate (datetime) - date the record was last updated
- UpdatedBy (varchar(256)) - the user that inserted or last updated the record
- ts (timestamp) - used for concurrency checks
- IsSetForDelete (bit) - flag for indirect deletes

#### DynamicFormFieldToEvent
Join table for associating fields to events.

Fields:
- DynamicFormFieldId (int) - foreign key to the [DynamicFormField](#dynamicFormField) table
- DynamicFormEventId (int) - foreign key to the [DynamicFormEvent](#dynamicFormEvent) table
- Callback (varchar(50)) - callback function name
- ArgumentName (varchar(50)) - name of the argument to pass to the callback function
- InsertedDate (datetime) - date the record was inserted
- UpdatedDate (datetime) - date the record was last updated
- UpdatedBy (varchar(256)) - the user that inserted or last updated the record
- ts (timestamp) - used for concurrency checks
- IsSetForDelete (bit) - flag for indirect deletes

## Record Mode
##### Multiple Record Mode
- displays a data grid with the applicable data.
- initially displays the form with all elements in a readonly/disabled state
- when a row in the data grid is clicked, the form will populate with the data from the selected record
- an "Create New [Name]" button is visible

##### Single Record Mode
- displays only the form with the form elements populated with the applicable data
- if more than one record is returned from the server, an informational error message appears stating that the Dynamic Form is in single-record mode, but multiple records were returned
- no "Create New [Name]" button is visible

## Form Layout
The rendered layout of html elements is determined by various configuration settings. The [Bootstrap](http://getbootstrap.com) framework is used for the layout and styling. The [Bootstrap](http://getbootstrap.com) framework uses a grid system that renders twelve(12) equal cells.

## Child Dynamic Forms
A child form is created through the configuration tables by setting the [dyn].[DynamicForm].[ParentId] field to the primary key value of the desired parent. A child form will be nested within the parent form container and will behave as any other form.

## Usage
Dynamic form usage begins with configuring the necessary data in the Dynamic Form tables. Once the data entry has been completed, you are ready to use the Dynamic Forms in your application. Whether you are using the Web plugin or the custom directive only, your application must load the javascript dependencies.

#### Web Plugin Usage
To integrate a Dynamic Form view into your AngularJS application, configure a [MenuItem] record in the AccessControl database for the desired subsystem with the following values:
- Text - The text to display in the menu item
- Route - /dynamicForm
- BaseName - dynamicForm
- Path - dynamicForm/
- Href - dynamicForm
- HrefParams - &dfn=[name] (name is the value in the dyn.DynamicForm.Name field)
- MenuActiveRoutes - /dynamicForm
- IsActive - true
- IsSecure - true/false (depends on your needs)
- IsTnRequired - true/false (depends on your needs)
- IsCdn - true (web plugins are delivered from the CDN)
- IsPlugin - true
- RenderInUi - true
- RequiresSubsystem - true
- ReloadOnSearch - true
- Rights - provide applicable rights
- ControllerAs - vm

You must then associate the [MenuItem] record created above the following [MenuItemDependency] records:
- cdn/plugin/dynamicForm/code/dynamicForm/dynamicFormService
- cdn/plugin/dynamicForm/code/dynamicForm/ecaDynamicFormSelector
- cdn/plugin/dynamicForm/code/dynamicForm/ecaDynamicFormDetail
- cdn/plugin/dynamicForm/code/dynamicForm/ecaDynamicFormSelectorChild
- cdn/plugin/dynamicForm/code/dynamicForm/deleteModalService
- cdn/plugin/dynamicForm/code/dynamicForm/ecaToBoolean
- cdn/plugin/dynamicForm/code/dynamicForm/ecaToNumber
- cdn/code/shared/filters/trueFalseFilter

#### Directive Only Usage
Dynamic Forms can be used by any AngularJS application by adding the custom directive "eca-dynamic-form-selector". Sample usage:
```
<eca-dynamic-form-selector form-id="loan" filter-id="{{vm.filterId}}"></eca-dynamic-form-selector>
```
In the sample above, the `form-id` attribute value is the name that corresponds with the [dyn].[DynamicForm].[Name] field and the `filter-id` attribute contains an angular expression. The value of the expression is used to filter the data returned by the API.

## Event Usage
Add the applicable data to the [dyn].[DynamicFormFieldToEvent] table. For example:
- set the foreign key fields [DynamicFormFieldId] and [DynamicFormEventId] for the field and event you wish to configure.
- set the Callback field value to the name of your controller's function to execute
- set the ArgumentName field value to the name of your callback functions argument. The argument will return an object with the following properties:
  - data: represents the data returned from the associated event. For example, if the html element type is select and the event is "ng-change", the data will be the selected data item
  - df: represents the Dynamic Form entity
  - eventName: "DynamicForm.[Name].[FieldName].[EventName]". For example "DynamicForm.Loan.SalesPerson.ng-change".
 
## Troubleshooting
- Multiple dynamic forms (web plugin)
  - If there are multiple dynamic forms used by an application, the [MenuItem] setup must be differentiated by setting [HrefParams] and [MenuActiveRoutes] to different values for each menu item by adding a query string parameter.
  - Loan
    - HrefParams: &dfn=loan&v=1
    - MenuActiveRoutes: /dynamicForm&v=1
  - AltAddress
    - HrefParams: &dfn=altaddress&v=2
    - MenuActiveRoutes: /dynamicForm&v=2
  

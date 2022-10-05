## Project

Basic resource to organize general user access for all collections registered in the project.
A project also acts as an umbrella container for one or multiple collections and has to be available before collection creation is possible.

## Collection

Basic resource to organize stored data.
Collections can be described with more detailed metadata by registering one or multiple Object/s as _"specification"_ of the Collection.

Collections can also be versioned on-demand with a version number following semantic versioning principles.
On creation of a collection version the collection a shallow cloned and immutable copy of the collection is created.
The version number is defined manually by the user who pinpoints the Collection.

## Object

Resource which stores the data in the storage system.
An Object can only be owned by one collection at a time but references can be created for multiple collections.
Depending on the context an Object can represent data or metadata.
Additionally, an Object has revisions

### State system

Objects in the storage have states.
These are used to indicate the status of an Object during its lifecycle.

<dl>
  <dt>INITIALIZING</dt>
  <dd>After Object creation but before Object finishing.</dd>

  <dt>AVAILABLE</dt>
  <dd>After Object finishing if everything succeeded.</dd>

  <dt>UNAVAILABLE</dt>
  <dd>E.g. while an Object is being updated.</dd>

  <dt>ERRORy/dt>
  <dd>If something went wrong e.g. data proxy endpoints are down.</dd>

  <dt>TRASH</dt>
  <dd>Object was deleted and waits to be removed by the garbage collector.</dd>
</dl>

## Label/Hook [optional]

Simple resource which represents a key-value pair which is directly associated with a Collection, Object or ObjectGroup.

**Labels**
: Used to describe short properties of the resource with which it is associated.

**Hooks**
: Used to reference (external) services which automatically validate/process/analyze/... the data.

## ObjectGroup [optional]

ObjectGroups are a secondary resource to organize Objects inside Collections.

# Internal Data Structure


## Mandatory hierarchical resources

<div class="flex-container" markdown>
  <div class="flex-item" markdown>
### :material-folder-multiple-outline: Project

A Project is the basic resource to organize general user access for stored data <!--and/or data to be stored--> (i.e. Objects). 
It also acts as an umbrella container for all other resources which means that every hierarchy has a Project as root. 
This directly implies that every project name has to be globally unique in the AOS universe.

You can also archive Projects which makes the Project and all its subresources immutable. 
This feature is useful e.g. if the stored data shall be used for any kind of permanent publication.
  </div>

  <div class="flex-item" markdown>
### :material-file-document-outline: Object

An Object is the resource which fundamentally stores the data in the backend storage system. Depending on the context an Object can represent data or metadata. It must be owned by at least one Project which means that every Object needs at least one Project as root in its hierarchy. Nonetheless, it can be flexibly shared with all other resources by creating the corresponding relation.

Additionally, an Object has revisions in contrast to the other resources. 
Once uploaded, an Object is immutable. Updates create new Objects that reference the original Object, resulting in a history of changes.
  </div>
</div>

## Optional hierarchical resources

<div class="flex-container" markdown>
  <div class="flex-item" markdown>
### :material-folder-outline: Collection

A Collection is the basic resource to organize stored data (i.e. Objects) inside Projects. Collections should consist a loose collection of Objects and/or Datasets.

Collections can also be snapshot with a version number following semantic versioning principles. On creation of a Collection snapshot, an immutable clone of the Collection and all its subresources gets created. The version number has to be provided manually by the user who initiates the Collection snapshot.
  </div>
  <div class="flex-item" markdown>
### :material-file-document-multiple-outline: Dataset

Datasets are a secondary hierarchy resource to organize Objects either inside Collections and/or Projects directly. A Dataset should consist of closely related Objects and should be used to combine data and metadata for easier access and organization.

Datasets can also be snapshot with a version number following semantic versioning principles. On creation of a Dataset snapshot, an immutable clone of the Dataset and all its subresources gets created. The version number has to be provided manually by the user who initiates the Dataset snapshot.
  </div>
</div>


## Other resources

<div class="flex-container" markdown>
  <div class="flex-item" markdown>
### :material-tag-text-outline: Label

Simple resource representing a plaintext key-value pair which is directly associated with an individual Project, Collection, Dataset or Object.
A Label can be used to describe short additional properties of a resource.
  </div>
  <div class="flex-item" markdown>
### :material-cog-sync-outline: Hook

Simple resource representing a plaintext value which is directly associated with an individual Project, Collection, Dataset or Object.
A Hook can be used to reference (external) services which automatically process/validate/etc. the uploaded data upon registration.
  </div>
</div>


## Resource relations concept

All resources and their relationships form a directed acyclic graph (DAG) with Projects as roots and Objects as leaves. 
Collections and Datasets can exist directly beneath Projects but only a Dataset and/or Objects can be created inside a Collection. 
This gives us the following possibilities to create a hierarchy for uploaded data:

* `Project` > `Collection` > `Dataset` > `Object`
* `Project` > `Collection` > `Object`
* `Project` > `Dataset` > `Object`
* `Project` > `Object`

In our model, we also distinguish __internal relations__ between AOS resources and __external relations__  which point to resources outside of AOS e.g. a DOI. 

Following there is a list of predefined internal relations:

* `BELONGS_TO` - Relation which describes resource hierarchy (`Project` > `Collection` > `Dataset` > `Object`)
* `ORIGIN` - Relation to original resource of clone
* `VERSION` - Relation to resource the version/revision was created from
* `METADATA` - Data :fontawesome-solid-arrow-right-arrow-left: Metadata relation
* `POLICY` - Relation to custom policy associated with the resource (currently not supported)

But you also have the possibility to create further, user-defined relations which are not limited in direction and/or meaning.

<figure markdown>
  ![Basic concept of the Aruna Object Storage data structure resource relations](aruna_resources.png#only-light){ align=center }
  ![Basic concept of the Aruna Object Storage data structure resource relations](aruna_resources.dark.png#only-dark){ align=center }
  <figcaption markdown>Hierarchical structure of AOS resources. Resources form a directed acyclic graph of __**belongs to**__ relationships (blue) with Projects as roots and Objects as leaves. Resources can also describe horizontal **__version__** relationships (orange), __**data/metadata**__ relationships (yellow) or even custom user-defined relationships (green).</figcaption>
</figure>


## State system

Objects in the storage have states.
These are used to indicate the status of an Object during its lifecycle.

**INITIALIZING**

: After Object creation/initialization but before Object finishing.

**VALIDATING**

: After Object finish while data validation is still running.

**AVAILABLE**

: After Object finishing/validation if everything succeeded.

**UNAVAILABLE**

: E.g. while all Dataproxy endpoints are unavailable which hold the Objects' data.

**ERROR**

: If something went wrong e.g. incomplete upload of data.

**DELETED**

: Object was deleted and remains only as data tombstone.

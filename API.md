# KBase Sample Service API

## Data structures

### Sample

Note that what this is called is up in the air. `Sample`, `Sampled object`, and the plurals of
both are in contention.

#### SampleNode

```
{
    "id": <sample node ID string that is unique within the sample>,
    "type": <one of "BioReplicate", "TechReplicate", or "SubSample">,
    "parent_id": <id of the parent of this node or null for a BioReplicate node>,
    "parent_index": <integer array index (see Sample below) or null for a BioReplicate node>,
    "metadata_unitless":
        {
            "metadata_key_1": "metadata_value_1",
                            ...
            "metadata_key_N": "metadata_value_N"
        },
    "metadata_with_units":
        {
            "metadata_key_1": ["metadata_value_1", "metadata_units_1"],
                            ...
            "metadata_key_N": ["metadata_value_N", "metadata_units_N"]
        },
    "metadata_unitless_user":
        {
            "metadata_key_1": "metadata_value_1",
                            ...
            "metadata_key_N": "metadata_value_N"
        },
    "metadata_with_units_user":
        {
            "metadata_key_1": ["metadata_value_1", "metadata_units_1"],
                            ...
            "metadata_key_N": ["metadata_value_N", "metadata_units_N"]
        },
    "name": <a free text user-supplied short name for the node. Optional.>
    "description": <a free text description of the node. Optional.>
}
```

We could get more specific here - this is the most generalized design. For example, we could
add a `chemical_species` field that is similar to `metadata_with_units`, or metadata specifically
for specifying the node's relationship to its parent.

TODO: A controlled vocabulary for the metadata keys, values, and units needs to be developed.

The `*_user` metadata variants allow a user to include arbitrary metadata outside of the
controlled vocabulary.

`parent_index` is redundant to `parent_id`, and exists only for convenience. `parent_id`
is the source of truth.

If `type` is `BioReplicate` then `parent_id` MUST BE `null`. Otherwise `parent_id` MUST NOT be
`null`.


#### Sample
```
{
    "id": <globally unique to KBase sample ID string>,
    "version": <integer version of the sample>,
    "external_source": 
        [<a string ID for an external data source, e.g. ENIGMA>,
         <a string ID for a data unit in an external data source.>
        ]
    "nodes": <list of SampleNodes>,
    "last_root_node_index": <integer index of the last root node in the sample>,
    "name": <a free text user-supplied short name for the sample. Optional.>
    "description": <a free text description of the sample. Optional.>
}
```

A `Sample` contains one or more trees of `SampleNode`s. The root nodes of all the trees MUST BE
first in `nodes` and `last_root_node_index` defines the last node in `nodes` that is a root node.
This is for convieniece only and the `SampleNode`s are the source of truth for which index contains
the last root node.

Each tree in the `Sample` has a `BioReplicate` `SampleNode` at its root. All other nodes in the
tree MUST BE `TechReplicate` or `SubSample` nodes.

`external_source` is optional.

## API

Requests are authenticated by including the header `Authorization: <kbase token>` in the request.

### Save a new sample

```
AUTHORIZATION REQUIRED
POST /api/v1/newsample/
{
    "sample": <Sample>,
}

RETURNS:
{
    "id": <the new sample ID>,
    "version": 1
}
```

The `id` and `version` fields are ignored if present.

The service verifies the structure of the sample trees are correct, the node IDs are unique,
the controlled vocabularies are correct, etc.

TODO: interactions with the workspace sample set such that saving the sample set ensures the
user should have access to the sample. 

* Have the API save a new sample set in a specified workspace
* Have the API create a new version of a specified sample set, adding the new sample

### Save a new version of a sample
```
AUTHORIZATION REQUIRED
PUT /api/v1/sample/<sample ID>/upa/<KBase UPA>
{
    "sample": <Sample>,
    "prior_version": <integer version of the prior version of the sample. Optional>
}

RETURNS:
{
    "id": <the sample ID>,
    "version": <the new version>
}
```

If the ID of the sample does not exist, the save fails.

`<KBase UPA>` is the UPA of the sample set that grants write access to the sample.

The service verifies the structure of the sample trees are correct, the node IDs are unique,
the controlled vocabularies are correct, etc.

`prior_version` allows for optimistic concurrency control. If specified, if the version of the
sample when saved would not be `prior_version` + 1, the save is aborted.

TODO: interactions with the workspace sample set such that saving the sample set ensures the user should have access to the new version of the sample.

* Have the API save a new sample set in a specified workspace
* Have the API create a new version of a specified sample set, adding the new sample

### View a sample
```
AUTHORIZATION OPTIONAL
GET /api/v1/sample/<sample ID>/upa/<KBase UPA>

RETURNS:
{
    "sample": <Sample>
}
```
`<KBase UPA>` is the UPA of the object (sample set or data object) that grants access to the
sample.

### TODO

Linking samples that represent the same physical thing together

Linking data to samples

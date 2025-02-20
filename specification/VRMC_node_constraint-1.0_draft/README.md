# VRMC_vrm

## Contributors

## Status

* Work in progress

## Dependencies

Written against the glTF 2.0 spec.

## Overview

This extension allows glTF to constrain the transform of a node by another node.

This extension defines three constraint types: Position Constraint, Rotation Constraint, and Aim Constraint.

## Constraints

A constraint can be one of **position constraint**, **rotation constraint**, or **aim constraint**.

### Sources

Each constraint specifies a source that will be used to be a reference transform when it constrains the destination node.

Nodes must meet these requirements to be a source:

- The source must not be the destination itself
- The source evaluated in model space (will be described in below) must not be a child of the destination
- The source must not make a circular dependency between two or more constraints

### Constraint spaces

Each constraint specifies two object spaces: **source space** and **destination space**.
Source space determines how to observe the transform of the source node.
Destination space determines how to apply the transform to the destination node.

Each space can be either of **local space** or **model space**.
When space is local space, the transform will be evaluated in the local of the node.
When space is model space, the transform will be evaluated relative from the root of the glTF scene.
Any transform can never be evaluated in world space.

### Position Constraint

A position constraint constrains the position of a node by another node.

The position of the source and the destination will be evaluated relative from their initial positions, meaning just applying the constraint without any translation of the source does not do anything to the source.

#### Freeze Axes

When the freeze axes is specified, the position of each frozen axis will be constrained by the constraint.
If an axis is not frozen, the constraint does not do anything to the position axis.

#### Weight

When the weight is specified, the position offset contributes to the destination node will be simply multiplied by the weight.

### Rotation Constraint

A rotation constraint constrains the rotation of a node by another node.

The rotation of the source and the destination will be evaluated relative from their initial rotations, meaning just applying the constraint without any rotation of the source does not do anything to the source.

> **TODO**: Rotation offset should be described more precisely

#### Freeze Axes

When the freeze axes is specified, the rotation of each frozen axis will be constrained by the constraint.
If an axis is not frozen, the constraint does not do anything to the orientation around the rotation axis.

> **TODO**: How do we "ignore" the orientation? The implementation should be described

#### Weight

The final contribution to the rotation offset will be determined by slerp of identity quaternion toward rotation offset of the source where t is the weight.

### Aim Constraint

An aim constraint rotates a node to make it look toward another node.

Rotation offset will be calculated using a **aim vector** and a **up vector** in the procedure below:

- Considering the destination node is facing toward the aim vector while its head faces toward a hemisphere around the up vector, calling it initial state
- Rotate the node to make it face toward the source node while up vector still faces toward the hemisphere around the up vector, calling it current state
- The rotation offset is the rotation between the initial state and the current state

The contribution to the destination will be a rotation from initial rotation offset toward current rotation offset, meaning just applying the constraint without any translaton of the source does not do anything to the source.

> **TODO**: Should we really use the initial roation offset?

#### Freeze Axes

In aim rotation, the property freeze axes freezes two axes: yaw and pitch.

When yaw is frozen, the destination node will be rotated around the up vector.
When pitch is frozen, the destination node will be rotated around the cross product of aim vector and up vector.

#### Weight

The final contribution to the rotation offset will be determined by slerp of identity quaternion toward the rotation offset where t is the weight.

---

## glTF Schema Updates

### Extending Nodes

A constraint will be described by adding a `VRMC_node_constraint-1.0` extension to a node.
For example, the following defines a constraint that constrains the position of the node `NodeB` by another node `NodeA`:

```json
{
  "nodes": [
    {
      "name": "NodeA",
    },
    {
      "name": "NodeB",
      "extensions": {
        "VRMC_node_constraint-1.0": {
          "position": {
            "source": 0,
            "weight": 1.0
          }
        }
      }
    }
  ]
}
```

---

### constraints

The root object of the extension.

#### Properties

|            | Type     | Description            | Required |
|:-----------|:---------|:-----------------------|:---------|
| `position` | `object` | A position constraint. | No       |
| `rotation` | `object` | A rotation constraint. | No       |
| `aim`      | `object` | An aim constraint.     | No       |

You must specify one of them and cannot specify more than one.

- JSON schema: [VRMC_node_constraint.schema.json](./schema/VRMC_node_constraint.schema.json)

#### constraints.position

A [position constraint](#positionConstraint).

- Type: `object`
- Required: No

#### constraints.rotation

A [rotation constraint](#rotationConstraint).

- Type: `object`
- Required: No

#### constraints.aim

An [aim constraint](#aimConstraint).

- Type: `object`
- Required: No

---

### positionConstraint

A set of parameters of a position constraint can be used to constrain the position of a node by another node.

#### Properties

|                    | Type         | Description                                             | Required                          |
|:-------------------|:-------------|:--------------------------------------------------------|:----------------------------------|
| `source`           | `integer`    | The index of the node constrains the node.              | ✅ Yes                           |
| `sourceSpace`      | `string`     | The source node will be evaluated in this space.        | No, default: `model`              |
| `destinationSpace` | `string`     | The destination node will be evaluated in this space.   | No, default: `model`              |
| `freezeAxes`       | `boolean[3]` | Axes be constrained by this constraint, in X-Y-Z order. | No, default: `[true, true, true]` |
| `weight`           | `number`     | The weight of the constraint.                           | No, default: `1.0`                |

- JSON schema: [VRMC_node_constraint.positionConstraint.schema.json](./schema/VRMC_node_constraint.positionConstraint.schema.json)

#### positionConstraint.source ✅

The index of the node constrains the node.

- Type: `integer`
- Required: Yes
- Minimum: `>= 0`

#### positionConstraint.sourceSpace

The source node will be evaluated in this space.

- Type: `string`
- Required: No, default: `model`
- Allowed values:
  - `local`
  - `model`

#### positionConstraint.destinationSpace

The destination node will be evaluated in this space.

- Type: `string`
- Required: No, default: `model`
- Allowed values:
  - `local`
  - `model`

#### positionConstraint.freezeAxes

Axes be constrained by this constraint, in X-Y-Z order.

- Type: `boolean[3]`
- Required: No, default: `[true, true, true]`

#### positionConstraint.weight

The weight of the constraint.

- Type: `number`
- Required: No, default: `1.0`

---

### rotationConstraint

A set of parameters of a rotation constraint can be used to constrain a rotation of a node by another node.

#### Properties

|                    | Type         | Description                                             | Required                          |
|:-------------------|:-------------|:--------------------------------------------------------|:----------------------------------|
| `source`           | `integer`    | The index of the node constrains the node.              | ✅ Yes                           |
| `sourceSpace`      | `string`     | The source node will be evaluated in this space.        | No, default: `model`              |
| `destinationSpace` | `string`     | The destination node will be evaluated in this space.   | No, default: `model`              |
| `freezeAxes`       | `boolean[3]` | Axes be constrained by this constraint, in X-Y-Z order. | No, default: `[true, true, true]` |
| `weight`           | `number`     | The weight of the constraint.                           | No, default: `1.0`                |

- JSON schema: [VRMC_node_constraint.rotationConstraint.schema.json](./schema/VRMC_node_constraint.rotationConstraint.schema.json)

#### rotationConstraint.source ✅

The index of the node constrains the node.

- Type: `integer`
- Required: Yes
- Minimum: `>= 0`

#### rotationConstraint.sourceSpace

The source node will be evaluated in this space.

- Type: `string`
- Required: No, default: `model`
- Allowed values:
  - `local`
  - `model`

#### rotationConstraint.destinationSpace

The destination node will be evaluated in this space.

- Type: `string`
- Required: No, default: `model`
- Allowed values:
  - `local`
  - `model`

#### rotationConstraint.freezeAxes

Axes be constrained by this constraint, in X-Y-Z order.

- Type: `boolean[3]`
- Required: No, default: `[true, true, true]`

#### rotationConstraint.weight

The weight of the constraint.

- Type: `number`
- Required: No, default: `1.0`

---

### aimConstraint

A set of parameters of an aim constraint can be used to rotate a node to make it look toward another node.

#### Properties

|                    | Type         | Description                                                 | Required                    |
|:-------------------|:-------------|:------------------------------------------------------------|:----------------------------|
| `source`           | `integer`    | The index of the node constrains the node.                  | ✅ Yes                     |
| `sourceSpace`      | `string`     | The source node will be evaluated in this space.            | No, default: `model`        |
| `destinationSpace` | `string`     | The destination node will be evaluated in this space.       | No, default: `model`        |
| `aimVector`        | `number[3]`  | An axis which faces the direction of its source.            | No, default: `[0, 0, 1]`    |
| `upVector`         | `number[3]`  | An up axis of the constraint.                               | No, default: `[0, 1, 0]`    |
| `freezeAxes`       | `boolean[2]` | Axes be constrained by this constraint, in Yaw-Pitch order. | No, default: `[true, true]` |
| `weight`           | `number`     | The weight of the constraint.                               | No, default: `1.0`          |

- JSON schema: [VRMC_node_constraint.aimConstraint.schema.json](./schema/VRMC_node_constraint.aimConstraint.schema.json)

#### aimConstraint.source ✅

The index of the node constrains the node.

- Type: `integer`
- Required: Yes
- Minimum: `>= 0`

#### aimConstraint.sourceSpace

The source node will be evaluated in this space.

- Type: `string`
- Required: No, default: `model`
- Allowed values:
  - `local`
  - `model`

#### aimConstraint.destinationSpace

The destination node will be evaluated in this space.

- Type: `string`
- Required: No, default: `model`
- Allowed values:
  - `local`
  - `model`

#### aimConstraint.aimVector

An axis which faces the direction of its source.

- Type: `number[3]`
- Required: No, default: `[0, 0, 1]`

#### aimConstraint.upVector

An up axis of the constraint.

- Type: `number[3]`
- Required: No, default: `[0, 1, 0]`

#### aimConstraint.freezeAxes

Axes be constrained by this constraint, in Yaw-Pitch order.

- Type: `boolean[2]`
- Required: No, default: `[true, true]`

#### aimConstraint.weight

The weight of the constraint.

- Type: `number`
- Required: No, default: `1.0`

## Implementation Notes

*This section is non-normative.*

### Dependency resolution between constraints

A constraint often depends on other constraints.
Constraints should be updated in correct orders under cases that has dependencies, to prevent constraints from referring transforms that is not updated yet.

Constraints can be depends on these nodes:

- Ancestors (until the root of the model) of the source, if the source space is model space
- The source
- Ancestors (until the root of the model) of the destination, if the destination space is model space

The pseudocode is an example of procedure how constraints should be updated:

```
let constraintsPending = empty set of Constraint
let constraintsDone = empty set of Constraint

function updateConstraint( constraint: Constraint )
  if not constraintsDone.has( constraint ) then
    if constraintPending.has( constraint ) then
      throw "Circular dependency detected"
    end if

    constraintsPending.add( constraint )
    foreach dependency in constraint.dependencies do
      updateConstraint( constraint )
    end foreach
    constraintsPending.delete( constraint )

    constraint.update()

    constraintsDone.add( constraint )
  end if
end function

function updateConstraints
  foreach constraint in constraints do
    updateConstraint( constraint )
  end foreach
end function
```

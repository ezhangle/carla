Measurements
============

Every frame the server sends a package with the measurements and images gathered
to the client. This document describes the details of these measurements.

Time-stamps
-----------

Since CARLA can be run at fixed-frame rate, we keep track of two different
time-stamps.

Key                        | Type      | Description
-------------------------- | --------- | ------------
platform_timestamp         | uint32    | Time-stamp of the current frame, in milliseconds as given by the OS.
game_timestamp             | uint32    | In-game time-stamp, milliseconds elapsed since the beginning of the current level.

In real-time mode, the elapsed time between two time steps should be similar
both platform and game time-stamps. When run in fixed-time step, the game
time-stamp increments in constant time steps (delta=1/FPS) while the platform
time-stamp keeps the actual time elapsed.

Player measurements
-------------------

Key                        | Type      | Description
-------------------------- | --------- | ------------
transform                  | Transform | World transform of the player.
acceleration               | Vector3D  | Current acceleration of the player.
forward_speed              | float     | Forward speed in km/h.
collision_vehicles         | float     | Collision intensity with other vehicles.
collision_pedestrians      | float     | Collision intensity with pedestrians.
collision_other            | float     | General collision intensity (everything else but pedestrians and vehicles).
intersection_otherlane     | float     | Percentage of the car invading other lanes.
intersection_offroad       | float     | Percentage of the car off-road.
ai_control                 | Control   | Vehicle's AI control that would apply this frame.

###### Transform

The transform contains two Vector3D objects, location and orientation.
Currently, the orientation is represented as the Cartesian coordinates X, Y, Z.
_We will probably change this in the future to Roll, Pitch, and Yaw_

###### Collision

Collision variables keep an accumulation of all the collisions occurred during
this episode. Every collision contributes proportionally to the intensity of the
collision (norm of the normal impulse between the two colliding objects).

Three different counts are kept (pedestrians, vehicles, and other). Colliding
objects are classified based on their tag (same as for semantic segmentation).

!!! Bug
    See [#13 Collisions are not annotated when vehicle's speed is low](https://github.com/carla-simulator/carla/issues/13)

Collisions are not annotated if the vehicle is not moving (<1km/h) to avoid
annotating undesired collision due to mistakes in the AI of non-player agents.

###### Lane/off-road intersection

The lane intersection measures the percentage of the vehicle invading the
opposite lane. The off-road intersection measures the percentage of the vehicle
outside the road.

These values are computed intersecting the bounding box of the vehicle (as a 2D
rectangle) against the map image of the city. These images are generated in the
editor and serialized for runtime use. You can find them too in the release
package under the folder "RoadMaps".

###### AI control

The `ai_control` measurement contains the control values that the in-game AI
would apply if it were controlling the vehicle.

This is the same structure used to send the vehicle control to the server.

Key                        | Type      | Description
-------------------------- | --------- | ------------
steer                      | float     | Steering angle between [-1.0, 1.0] (*)
throttle                   | float     | Throttle input between [ 0.0, 1.0]
brake                      | float     | Brake input between [ 0.0, 1.0]
hand_brake                 | bool      | Whether the hand-brake is engaged
reverse                    | bool      | Whether the vehicle is in reverse gear

(*) The actual steering angle depends on the vehicle used. The default Mustang
has a maximum steering angle of 70 degrees (this can be checked in the vehicle's
front wheel blueprint).

Non-player agents info
----------------------

To receive info of every non-player agent in the scene every frame you need to
activate this option in the settings file sent by the client at the beginning of
the episode.

```ini
[CARLA/Server]
SendNonPlayerAgentsInfo=true
```

If enabled, the server attaches a list of agents to the measurements package
every frame. Each of these agents has an unique id that identifies it, and
belongs to one of the following classes

  * **Vehicle** Contains its transform, bounding-box, and forward speed.
  * **Pedestrian** Contains its transform, bounding-box, and forward speed. (*)
  * **Traffic light** Contains its transform and state (green, yellow, red).
  * **Speed-limit sign** Contains its transform and speed-limit.

(*) At this point every pedestrian is assumed to have the same bounding-box
size.

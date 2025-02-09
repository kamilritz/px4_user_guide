# Return Mode

[<img src="../../assets/site/position_fixed.svg" title="Position fix required (e.g. GPS)" width="30px" />](../getting_started/flight_modes.md#key_position_fixed)

The *Return* flight mode is used to *fly a vehicle to safety*  on an unobstructed path to a safe destination, where it may either wait (hover or circle) or land.

PX4 provides several mechanisms for choosing a safe return path, destination and landing, including using safe points, mission paths, and mission landing sequences.

The following sections explain how to configure the [return type](#return_types), [return altitude](#return_altitude) and [landing/arrival behaviour](#arrival).
At the end there are sections explaining the *default* (preconfigured) behaviour for each [vehicle type](#default_configuration).

> **Note** 
> * This mode requires GPS.
> * This mode is automatic - no user intervention is *required* to control the vehicle.
> * RC control switches can be used to change flight modes on any vehicle.
    The effect of RC stick movement depends on the vehicle type.


## Return Types (RTL_TYPE) {#return_types}

PX4 allows you to specify one of three different approaches for finding an unobstructed path to a safe destination and/or landing:
- [Safety point return](#safety_point_return) ([RTL_TYPE=0](#RTL_TYPE)): Ascend to safe altitude and return to a safe point (home location)
- [Mission landing return](#mission_landing_return) (`RTL_TYPE=1`): Ascend to a safe altitude, fly direct to start of mission landing (if defined) and land.
- [Mission path return](#mission_path_return) (`RTL_TYPE=2`): Use mission path and continue to mission landing (if defined). If no mission landing defined, reverse mission to home.

For the mission based return types, the vehicle will fall back to a safe point return if no mission/mission landing is defined.

More detailed explanations for each of the types are provided in the following sections.

### Safety Point Return Type (RTL_TYPE=0) {#safety_point_return}

In this return type the vehicle:
- Ascends to a safe [return altitude](#return_altitude) (above any expected obstacles).
- Flies to safe point (home position) via direct path
- [Land or waits](#arrival) at descent altitude

> **Note** This return type is the fallback for the other types if no mission or mission landing is defined.

<span></span>
> **Note** This return type is commonly referred to as *Return to Launch* (RTL) and *Return to Home* (RTH).


### Mission Landing Return Type (RTL_TYPE=1) {#mission_landing_return}

This return type uses a *mission landing pattern* (if defined) to set the return destination and landing behaviour.

> **Note** A mission landing pattern consists of a [MAV_CMD_DO_LAND_START](https://mavlink.io/en/messages/common.html#MAV_CMD_DO_LAND_START), one or more position waypoints, and a [MAV_CMD_NAV_LAND](https://mavlink.io/en/messages/common.html#MAV_CMD_NAV_LAND).

If a mission landing has been defined, the vehicle will:
- Ascend to a safe [return altitude](#return_altitude) above any expected obstacles.
- Fly a direct path to the start of a mission landing sequence
- Descend/ascend to the specified waypoint altitude
- Execute the landing sequence.

If no mission landing is defined PX4 will revert to a [Safety Point Return](#safety_point_return).


### Mission Path Return Type (RTL_TYPE=2) {#mission_path_return}

This return type uses the mission (if defined) to provide a safe return *path*, and the mission landing pattern (if defined) to provide landing behaviour.
If there is a mission but no mission landing pattern, the mission is flown *in reverse*.

> **Note** The behaviour is fairly complex because it depends on the flight mode, and whether a mission and mission landing are defined.

Mission *with* landing pattern:
- **Mission mode:** Mission is continued in "fast-forward mode" (jumps, delay and any other non-position commands ignored, loiter and other position waypoints converted to simple waypoints) and then lands.
- **Auto mode other than mission mode:**
  - Ascend to a safe [return altitude](#return_altitude) above any expected obstacles.
  - Fly directly to closest waypoint (for FW not a landing WP) and descend to waypoint altitude.
  - Continue mission in fast forward mode from that waypoint.
- **Manual modes:** Same as [Mission Landing Return](#mission_landing_return).
  - Ascend to a safe [return altitude](#return_altitude) above any expected obstacles.
  - Fly directly to landing sequence position and descend to waypoint altitude
  - Land using mission landing pattern

Mission *without* landing pattern defined:
- **Mission mode:** 
  - Mission flown "fast-backward" (in reverse) starting from the previous waypoint 
    - Jumps, delay and any other non-position commands ignored, loiter and other position waypoints converted to simple waypoints.
    - VTOL vehicles transition to FW mode (if needed) before flying the mission in reverse.
  - On reaching waypoint 1, the vehicle ascends to the [return altitude](#return_altitude) and flies to the home position (where it [lands or waits](#arrival)).
- **Auto mode other than mission mode:**
  - Fly directly to closest waypoint (for FW not a landing WP) and descend to waypoint altitude.
  - Continue the mission in reverse, exactly as though RTL was triggered in mission mode (above)
- **Manual modes:** Same as [Safety Point Return](#safety_point_return).

If no mission is defined PX4 will revert to a [Safety Point Return](#safety_point_return).

If the mission changes during return mode, then the behaviour is re-evaluated based on the new mission following the same rules as above (e.g. if the new mission has no landing sequence and you're in a mission, the mission is reversed). 


## Return Altitude {#return_altitude}

A vehicle will usually first ascend to a safe altitude before returning (in order to avoid any obstacles between it and the destination).

> **Note** The exception is when executing a [mission path return](#mission_path_return) from within a mission.
  In this case the vehicle follows mission waypoints, which we assume avoid any obstacles).

The return altitude can be configured using the parameters [RTL_RETURN_ALT](#RTL_RETURN_ALT) and [RTL_CONE_ANG](#RTL_CONE_ANG) (which define a half cone centered around the home position).

![Return mode cone](../../assets/flying/rtl_cone.jpg)

<!-- Original draw.io diagram can be found here: https://drive.google.com/file/d/1W72XeZYSOkRlBSbPXCCiam9NMAyAWSg-/view?usp=sharing -->

If the vehicle is:
- Above [RTL_RETURN_ALT](#RTL_RETURN_ALT) (1) it will return at its current altitude.
- Below the cone it will return where it intersects the cone (2) or [RTL_DESCEND_ALT](#RTL_DESCEND_ALT) (whichever is higher).
- Outside the cone (3) it will first climb until it reaches [RTL_RETURN_ALT](#RTL_RETURN_ALT).
- Inside the cone:
  - Above [RTL_DESCEND_ALT](#RTL_DESCEND_ALT) (4) it will return at its current altitude.
  - Below [RTL_DESCEND_ALT](#RTL_DESCEND_ALT) (5) it will first ascend to `RTL_DESCEND_ALT`.

Note:
- If [RTL_CONE_ANG](#RTL_CONE_ANG) is 0 degrees there is no "cone":
  - the vehicle returns at `RTL_RETURN_ALT` (or above).
- If [RTL_CONE_ANG](#RTL_CONE_ANG) is 90 degrees the vehicle will return at the greater of `RTL_DESCEND_ALT` and the current altitude.
- The vehicle will always ascend at least [RTL_DESCEND_ALT](#RTL_DESCEND_ALT) for the return.


RC stick movement will [by default](#COM_RC_OVERRIDE) change the vehicle to [Position mode](../flight_modes/position_mc.md) unless handling a critical battery failsafe.


## Hover/Landing at Destination {#arrival}

Unless executing a mission landing (e.g. if executing a [safety point return](#safety_point_return)), the vehicle will arrive at its destination, and rapidly descend to the [RTL_DESCEND_ALT](#RTL_DESCEND_ALT) altitude.

The vehicle will the loiter for a specified time ([RTL_LAND_DELAY](#RTL_LAND_DELAY)) and then land.
If [RTL_LAND_DELAY=-1](#RTL_LAND_DELAY) it will loiter indefinitely.


## Vehicle Default Behaviour {#default_configuration}

The mode is _implemented_ in almost exactly the same way in all vehicle types (the exception being that fixed wing vehicles will circle rather than hover when waiting), and are hence tuned using the same parameters.

However the *default configuration* is tailored to suit the vehicle type, as described below.

### Multi-Copter (MC)

Multicopters use a [safety point return type](#safety_point_return) by default (and the following configuration):
- Ascend to [RTL_RETURN_ALT](#RTL_RETURN_ALT) ([RTL_CONE_ANG=0](#RTL_CONE_ANG) - cone not used).
- Fly to the home position in a straight line and constant altitude (if already above the return altitude it will return at its current altitude).
- Rapidly descend to the [RTL_DESCEND_ALT](#RTL_DESCEND_ALT) altitude.
- Land more or less immediately (small [RTL_LAND_DELAY](#RTL_LAND_DELAY)).


### Fixed Wing (FW)

Fixed-wing aircraft use a [mission landing return type](#mission_landing_return) by default: 
- If a mission landing is defined, fly direct to the mission landing start point and then land.
- Otherwise fly directly to the home position and circle above it at radius [NAV_LOITER_RAD](#NAV_LOITER_RAD).

If not following a mission landing, and [RTL_LAND_DELAY](#RTL_LAND_DELAY) is set to -1, the vehicle will land as described in the topic: [Landing (Fixed Wing)](../flying/fixed_wing_landing.md).
RC stick movement is ignored.


### VTOL

VTOL aircraft use a [mission landing return type](#mission_landing_return) by default: 
- If a mission landing is defined, fly direct to the mission landing start point and then land.
- Otherwise fly directly to the home position, transition to multicopter mode, and land as a multicopter.
  > **Note** If not in a mission landing, a VTOL in FW mode will *always* transition back to MC just before landing (ignoring [NAV_FORCE_VT](../advanced_config/parameter_reference.md#NAV_FORCE_VT)).

## Parameters

The RTL parameters are listed in [Parameter Reference > Return Mode](../advanced_config/parameter_reference.md#return-mode) (and summarised below).

Parameter | Description
--- | ---
<span id="RTL_TYPE"></span>[RTL_TYPE](../advanced_config/parameter_reference.md#RTL_TYPE) | Return mechanism (path and destination). `0`: Return to safe point via direct path, `1`: Use mission landing if defined (otherwise type `0`), `2`: Use mission path either forward to landing (if landing defined) or reversed to home (or type `0` if no mission).
<span id="RTL_RETURN_ALT"></span>[RTL_RETURN_ALT](../advanced_config/parameter_reference.md#RTL_RETURN_ALT) | Return altitude in meters (default: 60m) when [RTL_CONE_ANG](../advanced_config/parameter_reference.md#RTL_CONE_ANG) is 0. If already above this value the vehicle will return at its current altitude.
<span id="RTL_DESCEND_ALT"></span>[RTL_DESCEND_ALT](../advanced_config/parameter_reference.md#RTL_DESCEND_ALT) | Minimum return altitude and altitude at which the vehicle will slow or stop its initial descent from a higher return altitude (default: 30m)
<span id="RTL_LAND_DELAY"></span>[RTL_LAND_DELAY](../advanced_config/parameter_reference.md#RTL_LAND_DELAY) | Time to hover at `RTL_DESCEND_ALT` before landing (default: 0.5s) -by default this period is short so that the vehicle will simply slow and then land immediately. If set to -1 the system will loiter at `RTL_DESCEND_ALT` rather than landing. The delay is provided to allow you to configure time for landing gear to be deployed (triggered automatically). 
<span id="RTL_MIN_DIST"></span>[RTL_MIN_DIST](../advanced_config/parameter_reference.md#RTL_MIN_DIST) | Minimum horizontal distance from home position to trigger ascent to the return altitude specified by the "cone". If the vehicle is horizontally closer than this distance to home, it will return at its current altitude or `RTL_DESCEND_ALT` (whichever is higher) instead of first ascending to RTL_RETURN_ALT).
<span id="RTL_CONE_ANG"></span>[RTL_CONE_ANG](../advanced_config/parameter_reference.md#RTL_CONE_ANG) | Half-angle of the cone that defines the vehicle RTL return altitude. Values (in degrees): 0, 25, 45, 65, 80, 90. Note that 0 is "no cone" (always return at `RTL_RETURN_ALT` or higher), while 90 indicates that the vehicle must return at the current altitude or `RTL_DESCEND_ALT` (whichever is higher).
<span id="COM_RC_OVERRIDE"></span>[COM_RC_OVERRIDE](../advanced_config/parameter_reference.md#COM_RC_OVERRIDE) | If enabled stick movement gives control back to the pilot in [Position mode](../flight_modes/position_mc.md) (except when vehicle is handling a critical battery failsafe). Enabled by default.
<span id="NAV_LOITER_RAD"></span>[NAV_LOITER_RAD](../advanced_config/parameter_reference.md#NAV_LOITER_RAD) | [Fixed Wing Only] The radius of the loiter circle (at [RTL_LAND_DELAY](#RTL_LAND_DELAY). 

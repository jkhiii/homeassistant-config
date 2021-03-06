# Life360
This platform allows you to detect presence using [Life360](http://life360.com/).

To use this custom component, place a copy of:

[life360.py](https://github.com/pnbruckner/homeassistant-config/blob/master/custom_components/life360.py) at `<config>/custom_components/life360.py` and

[device_tracker/life360.py](https://github.com/pnbruckner/homeassistant-config/blob/master/custom_components/device_tracker/life360.py) at `<config>/custom_components/life360.py`

where `<config>` is your Home Assistant configuration directory. Then add the desired configuration. Here is an example of a typical configuration:
```yaml
device_tracker:
  - platform: life360
    username: !secret life360_username
    password: !secret life360_password
    show_as_state: places, moving
    max_update_wait:
      minutes: 30
```
## Configuration variables:

- **username**: Your Life360 username.
- **password**: Your Life360 password.
- **interval_seconds** (*Optional*): The default is 12. This defines how often the Life360 server will be queried. The resulting device_tracker entities will actually only be updated when the Life360 server provides new location information for each member.
- **filename** (*Optional*): The default is life360.conf. The platform will get an authorization token from the Life360 server using your username and password, and it will save the token in a file in the HA config directory (with limited permissions) so that it can reuse it after restarts (and not have to get a new token every time.) If the token eventually expires, a new one will be acquired as needed.
- **show_as_state** (*Optional*): Without it entities' states will be strictly determined by the device_tracker component. If specified must be one or more of: `places`, `driving` and `moving`. If `places` is specified then whenever Life360 reports a member is in a Life360 defined Place, the name of that Place will become the state of the device_tracker entity. If `driving` is specified and Life360 reports isDriving as true, then the entity's state will be 'Driving'. If `moving` is specified and Life360 reports inTransit is true, then the entity's state will be 'Moving'. If multiple options are specified and more than one becomes true at the same time, `driving` takes precedence over `moving`, being in a HA zone takes precedence over those two, and `places` takes precedence over all the others.
- **max_update_wait** (*Optional*): If you specify it, then if Life360 does not provide an update for a member within that maximum time window, the life360 platform will fire an event named `device_tracker.life360_update_overdue` with the entity_id of the corresponding member's device_tracker entity. Once an update does come it will fire an event named `device_tracker.life360_update_restored` with the entity_id of the corresponding member's device_tracker entity and another data item named `wait` that will indicate the amount of time spent waiting for the update. You can use these events in automations to be notified when they occur. See example automations below.
- **prefix** (*Optional*): Default is to name entities `device_tracker.<first_name>_<last_name>`, where `<first_name>` and `<last_name>` are specified by Life360. If a prefix is specified, then entity will be named `device_tracker.<prefix>_<first_name>_<last_name>`.
- **members** (*Optional*): Default is to track all Life360 Members in all Circles. If you'd rather only track a specific set of members, then list them with each member specified as `first,last`. If the member only has a first or last name in Life360, then enter as `first,` or `,last` (i.e., include the comma.) Names are case insensitive, and extra spaces are ignored.
## Example full configuration
```yaml
device_tracker:
  - platform: life360
    username: !secret life360_username
    password: !secret life360_password
    interval_seconds: 10
    filename: life360.conf
    show_as_state: places, driving, moving
    max_update_wait:
      minutes: 30
    prefix: life360
    members:
      - mike, smith
      - Joe,
      - , Jones
```
## Example overdue update automations
```yaml
- alias: Life360 Overdue Update
  trigger:
    platform: event
    event_type: device_tracker.life360_update_overdue
  action:
    service: notify.email_me
    data_template:
      title: Life360 update overdue
      message: >
        Update for {{
          state_attr(trigger.event.data.entity_id, 'friendly_name')
        }} is overdue.

- alias: Life360 Update Restored
  trigger:
    platform: event
    event_type: device_tracker.life360_update_restored
  action:
    service: notify.email_me
    data_template:
      title: Life360 update restored
      message: >
        Update for {{
          state_attr(trigger.event.data.entity_id, 'friendly_name')
        }} restored after {{ trigger.event.data.wait }}.
```

# Shared Line Appearance (SLA)

Legacy PBXes often had a feature called **Shared Line Appearance** for incoming calls. The functionality basically allowed multiple phones to have a line key tied to an extension or DID and offered several features:

- Incoming calls would ring all configured phones simultaneously
- When a phone answered the call, the status light would turn green; on the other phones, the status light would turn red
- If the caller is placed on hold, all phones' (answering or not) status lights will blink red
- Any phone with SLA can pickup the on-hold call
- When the call completes, all status lights turn off.

## Simulating SLA

Simulating most of SLA on a Kazoo system is possible. Let's take a look at how this might be achieved.

### Incoming calls ring all devices

This is pretty straight-forward. In Kazoo, one of the callflow actions is a [ring group](https://docs.2600hz.com/dev/applications/callflow/doc/ring_group/). Put any endpoints ([`devices`](https://docs.2600hz.com/dev/applications/crossbar/doc/devices/), [`users`](https://docs.2600hz.com/dev/applications/crossbar/doc/users/), or [`groups`](https://docs.2600hz.com/dev/applications/crossbar/doc/groups/)) into the ring group and all calls to the callflow will ring the endpoints according to the strategy provided.

### Status lights

Kazoo allow updating a pre-defined `presence_id` to with BLF updates. You can set this `presence_id` on the `device` or `user` documents, or you can use the `manual_presence` callflow action to toggle this. Configure the phones' BLF light to subscribe for this `presence_id` value to have it update during calls.

#### Placing on hold for others

The closest way to put a call on hold so others could pick it up is using [`call parking`](https://docs.2600hz.com/dev/applications/callflow/doc/park/). Rather than using the hold button on the phone, the answering device can transfer the caller to a parking slot. Once parked, any device that knows the slot number can pick up the call.

## Example SLA simulation

Consider the typical executive with an administrative assistant. The assistant's phone has a line key that will ring when the executive is called. The easiest way to do this is create two devices in Kazoo and assign them to the executive's Kazoo user.

### Create the executive

1. [Create a user](https://docs.2600hz.com/dev/applications/crossbar/doc/users/) for the executive. Be sure to include `presence_id="EXT"` in the user object - this is what BLF lights will be tied to when setting up presence.
2. [Create a device](https://docs.2600hz.com/dev/applications/crossbar/doc/devices/) for the executive.
3. [Create the callflow](https://docs.2600hz.com/dev/applications/crossbar/doc/callflows/) using the [ring group](https://docs.2600hz.com/dev/applications/callflow/doc/ring_group/) action to ring both the executive's and assistant's phones. This will also cause a BLF update to `presence_id`@`account.realm` to update the assistant's BLF key.

### Create the assistant

1. Create the user/device/callflow for the assistant (ringing the assistant's user in the callflow).

### Create the call park/pickup callflow

Create a callflow, `*4{presence_id}`, that will have the [`park`](https://docs.2600hz.com/dev/applications/callflow/doc/park/) callflow action. This will be used by the BLF key to simulate the hold/pickup of SLA. You should set the `action` to `auto` and the `slot` to `presence_id`.

### Create the BLF key

Create a BLF key configuration on the assistant's phone to:

a. The "value" (or equivalent) will be `presence_id`
b. The "extension" (or equivalent) will be `*4{presence_id}`

This will light up with the corresponding call.

When the assistant puts the call on hold, the BLF can be used to pickup the call.

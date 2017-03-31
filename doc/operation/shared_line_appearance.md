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

Consider an office with 3 phones and a DID to be shared among them.

### Status lights

First, we need a `presence_id` to represent the shared line. Perhaps the last four digits of the DID would work? Let's use `3456` for our purposes.

This `presence_id` will tie together the ring group and the parking. This value is what the BLF light on the phones will be set to `{presence_id}@account.realm` or `3456@account.realm` in this case.

### Ring group

The ring group actually needs a little help to make sure the presence updates are being set. The high level callflow looks like:

[DID] -> [Manual Presence: ringing] -> [Ring Group] -> [



```shell
curl -X PUT -H "X-Auth-Token: $AUTH_TOKEN" \
     -d "{\"data\": {\"flow\": { \"data\": { \"endpoints\": [{\"endpoint_type\": \"device\",\"id\": \"DEVICE_1_ID\"},{\"endpoint_type\": \"device\",\"id\": \"DEVICE_2_ID\"},{\"endpoint_type\": \"device\",\"id\": \"DEVICE_3_ID\"}]},\"module\": \"ring_group\"},\"name\": \"Main Number\",\"numbers\": [\"{DID}\"]}}" \
     http://ip.add.re.ss:8000/v2/accounts/$ACCOUNT_ID/callflows | python -mjson.tool
```

Great! Now every call to `{DID}` will ring all three devices simultaneously.

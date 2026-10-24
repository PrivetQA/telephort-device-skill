---
name: telephort-device
description: >-
  Create, start, and adjust a Telephort device the way the create form does,
  including internet checks, GPU slots, keyboxes, and optional Play Store
  update or Play Store sign-in. Use when the user asks to create a device,
  pick a proxy or VPN, update Play Store apps, or sign into Play Store.
compatibility: Any agent with the Telephort TypeScript or Python SDK. Create, update, and delete go through that REST client, not MCP.
---

# Telephort device

Speak the user's language. Do the steps below in order. Do not open Play Store unless this skill says to.

Use the Telephort SDK in `sdk/typescript` (`TelephortClient` from `@telephort/typescript-sdk`) or `sdk/python` (`telephort_sdk.TelephortClient`). The client needs `baseUrl`, `teamId`, and an access token (`auth.login` or `setAccessToken` / `set_access_token`). Do not create, update, or delete a device through MCP.

Typed calls below use the TypeScript names. Python uses the same objects with underscores: `devices.create`, `devices.update`, `devices.delete`, `devices.start`, `devices.stop`, `proxies.list`, `proxies.check`, `proxies.check_one_off`, `proxies.create`, `teams.get`. Paths that have no typed method use `request`.

## 1. Limits, only when nothing can be created

Read limits with `teams.get(teamId)`: `max_total_devices`, `max_running_devices`, `max_running_gpu_devices`, `max_running_cpu_devices`. List devices with `devices.list`, following `next_cursor`, `limit` 100.

Count Total from devices whose status is not `error` or `unknown`. Count Running, GPU(+), and GPU(-) from statuses `running`, `starting`, `provisioning`, and `stopping`. GPU(+) is `gpu_accel: true`. A kind's free slots are the minimum of that kind's remaining slots and Running's remaining slots.

- Nothing can be created when Total is full, Running is full, or both GPU kinds are full. Say which. Name the devices that hold it. Offer to stop a running device (`devices.stop`) or delete one that counts toward Total (`devices.delete`). Do that only after the user names the device and agrees. Count again. Do not continue until some device can be created and started.
- One of GPU(+) or GPU(-) being full is not a reason to stop here. Say that later, when the user chooses that kind.

A device in error or unknown does not free Total when deleted. Stopping a running device frees Running and its GPU(+) or GPU(-).

## 2. Internet

Ask for the internet before the model. The choices are Direct, a saved proxy, a saved VPN, or a new proxy or VPN the user pastes. List saved proxies with `proxies.list`. List saved VPNs with `request('GET', '/vpns')`.

Check before filling fields. Do not create on a failed check.

- Saved proxy: `proxies.check(id)`. Pasted proxy: `proxies.checkOneOff({ url, dns_ip })`.
- Saved VPN: `request('POST', '/vpns/{id}/check')`. Pasted VPN: `request('POST', '/vpns/check', { config, username, password, dns_ip })`.
- Direct: do not check a proxy. The place is Madrid, latitude `40.4167`, longitude `-3.7037`, country `ES`, moved 1–5 km from that point.

On success, fill the create-form fields and keep them. The user may later correct name, place, language, Wi-Fi or mobile, operator, and SIM. Do not replace a model they already chose.

- Name: `[CC] City Wi-Fi` or `[CC] City Mobile`, from the check city and `mobile`.
- Transport: `network_mode` `mobile` when `mobile` is true, otherwise `wifi`.
- Proxy or VPN coordinates stay exact. DNS is `77.88.8.8` for RU, otherwise `1.1.1.1`.
- Language comes from the country. Use `en_US` when the country has no locale.
- Operators: `request('GET', '/device-models/operators', { lat, lon })`. On mobile, match the operator to the check ISP or org; otherwise pick one. Build a valid SIM for that operator. If `operator_id` or `sim_msisdn` cannot be filled, do not create. Say the location has no operator and ask for another internet.
- Region: `request('GET', '/device-models/regions')`. Use it only when the list has one region. Otherwise ask.

Failed check:

- DNS failure: the proxy stays. Ask for another DNS. Do not ask for a new proxy yet.
- Unreachable, timeout, or rejected password: do not create. Say that reason. Ask for another internet. Do not switch to Direct.
- Retry a failed check once. Then ask the user again.

A new proxy or VPN is saved only if the user asks. Proxy: `proxies.create({ name, url, dns_ip })`. VPN: `request('POST', '/vpns', { name, config, username, password, dns_ip })`. Then create the device with the returned id.

## 3. Model

List models with `request('GET', '/device-models')`, following `next_cursor`, `limit` 100. Ask which model.

- `device_type: physical`: do not ask GPU or keybox. GPU is off and there is no keybox.
- `device_type: virtual`: ask GPU, then keybox.

GPU is for apps with a lot of video: TikTok, Facebook, Instagram. The usual choice is off. Ask. When they choose with or without GPU, use the counts from section 1.

If that kind has no room, say the limit is full. Offer only to stop or delete an existing device of that kind. Do not suggest switching GPU. After they agree, do it, count again, and continue only when that kind has a free slot and Total and Running still have room.

Keybox, virtual only. `request('GET', '/keyboxes')`, following `next_cursor`. Offer a Telephort service keybox (`is_service: true`), a team keybox if they have one, or none. Hide a rejected service keybox. If several exist, list them by name. Do not ask for a new keybox file unless the user says they already have one.

## 4. Fields, Play offers, then create

Show the filled fields and let the user correct name, place, language, transport, operator, and SIM.

Before create, offer both, as two separate questions:

- Обновить приложения в Play Store?
- Залогиниться в Play Store?

Ask for the Google account only if they agreed to sign in. If they did not, do not open a Google sign-in later.

Show the whole device once: model, internet, GPU, keybox, and the fields. Call `devices.create` only after they agree.

`gpu_accel` must be `false` unless they chose GPU on a virtual model. Send `false` when it is off. Do not omit it. Do not send `keybox_id` for a physical model. Do not send model, region, GPU, or keybox later to change them.

## 5. Start, then Play

Call `devices.start`.

Play Store package is `com.android.vending`. Open the screen with `connectDeviceAutomation` (Python: `connect_device_automation`) only for a step they agreed to.

- Both update and sign-in: open Play Store, update the apps, and only after that update sign in with the Google account. Ask for the code when Google shows it.
- Only update: open Play Store and update. If Play asks for a sign-in, stop. Do not sign in.
- Only sign-in: open Play Store and sign in. Do not update.
- No agreement: do not open Play Store.

To update: tap the `...` icon at the top right, then tap Update. Wait until the updates settle before any sign-in. About 5 apps left not updated is normal. 20 or fewer left is success. More than 20 left is not done: say so and do not pretend the apps are current. If `...` or Update is missing, stop and say that. Do not treat a missing button as "already updated".

## Edit and delete

`devices.update` changes internet, name, place, language, transport, operator, and SIM on a stopped device. It does not change model, region, GPU, or keybox. Pass the fields the user gave. When internet changes and name, place, language, transport, operator, or SIM was omitted, fill only those omitted fields from a new check, then PATCH. A running device must be stopped with `devices.stop` after the user agrees.

`devices.delete` only after the user agrees, and only when the device is stopped, in error, or unknown. Do not delete a running device and do not stop it as part of delete unless they agreed to stop it first.

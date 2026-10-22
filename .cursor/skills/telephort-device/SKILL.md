---
name: telephort-device
description: >-
  Create, start, and adjust a Telephort device the way the create form does,
  including internet checks, GPU slots, keyboxes, and optional Play Store
  update or Play Store sign-in. Use when the user asks to create a device,
  pick a proxy or VPN, update Play Store apps, or sign into Play Store.
---

# Telephort device

Speak the user's language. Do the steps below in order. Do not open Play Store unless this skill says to.

MCP tools: `device.quotas`, `device.models.list`, `device.proxies.list`, `device.vpns.list`, `device.keyboxes.list`, `device.calculate`, `device.create`, `device.start`, `device.stop`, `device.delete`, `device.get`, `device.update`. A live screen uses `session.open` and the action tools.

## 1. Limits, only when nothing can be created

Call `device.quotas` first.

- `can_create_any` false: say which of Total, Running, or both GPU kinds is full. Name the devices that hold it. Offer to stop a running device or delete one that counts toward Total. Do that only after the user names the device and agrees. Call `device.quotas` again. Do not continue until `can_create_any` is true.
- One of GPU(+) or GPU(-) being full is not a reason to stop here. Say that later, when the user chooses that kind.

A device in error or unknown does not free Total when deleted. Stopping a running device frees Running and its GPU(+) or GPU(-).

## 2. Internet

Ask for the internet before the model. The choices are Direct, a saved proxy, a saved VPN, or a new proxy or VPN the user pastes. List saved ones with `device.proxies.list` and `device.vpns.list`.

Call `device.calculate`. It does not create a device.

- `ok: true`: keep `fields`. The user may later correct name, place, language, Wi-Fi or mobile, operator, and SIM. Do not replace a model they already chose. If `operator_id` or `sim_msisdn` is missing, do not create. Say the location has no operator and ask for another internet.
- `PROXY_DNS_FAILED`: the proxy stays. Ask for another DNS. Do not ask for a new proxy yet.
- `PROXY_UNREACHABLE`, `PROXY_TIMEOUT`, `PROXY_AUTH_FAILED`, or `CHECK_FAILED`: do not create. Say the reason from the tool. Ask for another internet. Do not switch to Direct yourself.
- Retry a failed check once. Then ask the user again.

A new proxy or VPN is saved to the team library only if the user asks. Then pass `save_to_library: true` and `library_name` to `device.create`.

## 3. Model

Call `device.models.list`. Ask which model.

- `device_type: physical`: do not ask GPU or keybox. GPU is off and there is no keybox.
- `device_type: virtual`: ask GPU, then keybox.

GPU is for apps with a lot of video: TikTok, Facebook, Instagram. The usual choice is off. Ask. When they choose with or without GPU, look at `gpu_plus` or `gpu_minus` from `device.quotas`.

If that kind has no room, say the limit is full. Offer only to stop or delete an existing device of that kind. Do not suggest switching GPU. After they agree, do it, check quotas again, and continue only when `slot` for that choice exists (`gpu_plus.available` or `gpu_minus.available`, and Total and Running still have room).

## 4. Fields, Play offers, then create

Show the calculated fields and let the user correct name, place, language, transport, operator, and SIM.

Before create, offer both, as two separate questions:

- Обновить приложения в Play Store?
- Залогиниться в Play Store?

Ask for the Google account only if they agreed to sign in. If they did not, do not open a Google sign-in later.

Show the whole device once: model, internet, GPU, keybox, and the fields. Call `device.create` only after they agree.

`gpu_accel` must be `false` unless they chose GPU on a virtual model. Omitting it must stay off. Do not send a keybox for a physical model.

## 5. Start, then Play

Call `device.start`.

Play Store package is `com.android.vending`. Open it only for a step they agreed to.

- Both update and sign-in: open Play Store, update the apps, and only after that update sign in with the Google account. Ask for the code when Google shows it.
- Only update: open Play Store and update. If Play asks for a sign-in, stop. Do not sign in.
- Only sign-in: open Play Store and sign in. Do not update.
- No agreement: do not open Play Store.

To update: tap the `...` icon at the top right, then tap Update. Wait until the updates settle before any sign-in. About 5 apps left not updated is normal. 20 or fewer left is success. More than 20 left is not done: say so and do not pretend the apps are current. If `...` or Update is missing, stop and say that. Do not treat a missing button as "already updated".

## Edit and delete

`device.update` changes internet, name, place, language, transport, operator, and SIM on a stopped device. It does not change model, region, GPU, or keybox. A running device must be stopped with `device.stop` after the user agrees.

`device.delete` only after the user agrees, and only when the device is stopped, in error, or unknown. Do not delete a running device and do not stop it as part of delete unless they agreed to stop it first.

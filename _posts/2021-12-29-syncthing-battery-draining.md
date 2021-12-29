---
title: Stop Syncthing battery draining on Android
layout: post
icon: fa-battery-empty
icon-style: solid
---

In [degoogling]({{ site.baseurl }}{% link _posts/2021-12-21-degoogling-android.md %}) post, I explained how I switch to CalyxOS. One great relief is that, as most google services are not running, I should expect a lower battery consumption and so, a longer battery life.
It was true, until I install Syncthing to sync my data between my devices instead of using centralized Google Drive.The service was draining my whole battery, beeing the only app which eat it.
As I rarely use Wifi networks, I need Syncthing to run over mobile data. The problem is that it runs in background, constantly checking for new data.

![before]({{site.baseurl}}/assets/images/easer/before.png){: width="300" }

The main [Syncthing](https://f-droid.org/en/packages/com.nutomic.syncthingandroid/) app from f-droid, doesn't provide much battery saving options.

- ``Syncthing Options / Disable RestartOnWakeUp`` : It is supposed to save battery, tried it without success.
- ``Running conditions / Disable run on battery`` : It would be a good solution if I could be able to force a run, even on battery, in emergency case.

[Syncthing fork](https://f-droid.org/en/packages/com.github.catfriend1.syncthingandroid/) is supposed to be a better solution.

The [README](https://github.com/Catfriend1/syncthing-android/tree/ed83b22596eb0b575cda7b3fd5b9a1c5704def14#readme) says that "Battery eater problem is fixed.", so I [asked the dev](https://github.com/Catfriend1/syncthing-android/issues/870) for more explainations, his answer was a bit light.
Despite the fact that the fork gives some finer options to deal with battery consumption, it allowed me to save no more than 2 or 3 battery hours.

The solution would be to sync only on specific conditions, to reduce the running syncing window. Luckily, Syncthing has an option to ``Respect Android parameter about Data Syncing``, and some automation tools exists.

# Easer automation tool

[Easer](https://f-droid.org/fr/packages/ryey.easer/) is an Android event driven automation tool.
I want my sync to be disabled when the screen is locked. When the screen is unlocked, I want to limit sync to 1 min.
As my main concern is battery consumption, I want to always sync when battery is charging.
I usually charge at home, which is the only place where I use Wifi, lets trigger it then.

Here's my configuration.

## Conditions

Conditions are long time events, based on states. I used it to check if the screen is unlocked or the battery is discharging.

![conditions]({{site.baseurl}}/assets/images/easer/conditions.png){: width="300" } ![unlocked]({{site.baseurl}}/assets/images/easer/unlocked.png){: width="300" } ![charging]({{site.baseurl}}/assets/images/easer/charging.png){: width="300" }

## Events

Events are what they are, short time changing states, used for exemple for timing.

![events]({{site.baseurl}}/assets/images/easer/events.png){: width="300" } ![timing]({{site.baseurl}}/assets/images/easer/timing.png){: width="300" }

## Profils

Profils are set of actions.

![profils]({{site.baseurl}}/assets/images/easer/profils.png){: width="300" } ![enable_sync]({{site.baseurl}}/assets/images/easer/enable_sync.png){: width="300" } ![disable_wifi]({{site.baseurl}}/assets/images/easer/disable_wifi.png){: width="300" } ![enable_sync_and_wifi]({{site.baseurl}}/assets/images/easer/enable_sync_and_wifi.png){: width="300" } ![disable_sync]({{site.baseurl}}/assets/images/easer/disable_sync.png){: width="300" }

## Scripts

Scripts link events and conditions to profils.

![scripts]({{site.baseurl}}/assets/images/easer/scripts.png){: width="300" } ![when_charging]({{site.baseurl}}/assets/images/easer/when_charging.png){: width="300" } ![when_not_charging]({{site.baseurl}}/assets/images/easer/when_not_charging.png){: width="300" } ![when_unlocked]({{site.baseurl}}/assets/images/easer/when_unlocked.png){: width="300" } ![disable_sync_when_locked]({{site.baseurl}}/assets/images/easer/disable_sync_when_locked.png){: width="300" } ![disable_sync_1min]({{site.baseurl}}/assets/images/easer/disable_sync_1min.png){: width="300" }

## Pivot

This is the global algorithm.

![pivot]({{site.baseurl}}/assets/images/easer/pivot.png){: width="600" }

# For 24 hours to ... 5 days

After one night, here's the estimated battery duration time. Syncthing doesn't even appear in the list, which is normal because it was not syncing at all.

![after]({{site.baseurl}}/assets/images/easer/after.png){: width="300" }

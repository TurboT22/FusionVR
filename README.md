# FusionVR

[![Unity](https://img.shields.io/badge/Unity-2022.3%2B-black?logo=unity)](https://unity.com)
[![Photon Fusion 2](https://img.shields.io/badge/Photon%20Fusion-2%20(Shared%20Mode)-00a1e0)](https://www.photonengine.com/fusion)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Meta%20Quest%20%7C%20PCVR-blueviolet)](#requirements)

A drop-in **Photon Fusion 2 (Shared Mode)** multiplayer framework for VR with the same API shape as fchb1239's
**PhotonVR**: static `Connect`, `JoinRandomRoom`, `JoinPrivateRoom`, `SetUsername`, `SetColour`, `SetCosmetic`
and a hierarchy-based cosmetics system. Proximity voice chat is sent **over Fusion itself**, so you need **no
Photon Voice AppId and use zero Voice CCUs**.

## Features

- PhotonVR-style static API (`using Fusion.VR;`)
- Networked head and hands with smooth remote interpolation, plus username, colour and cosmetics for late joiners
- Hierarchy-based cosmetics: any child named `...Cosmetics...` under `Head`, `Face`, `Body`, `LeftHand` or `RightHand`
- Spatial proximity voice over Fusion reliable data (no external voice server)
- Physical VR buttons and a room-code keypad
- Editor tooling: Setup Wizard, prefab generator, custom inspectors, Cosmetic Helper

## Requirements

| Dependency | Notes |
|---|---|
| Unity 2022.3+ | |
| [Photon Fusion 2](https://doc.photonengine.com/fusion/current/getting-started/sdk-download) | Import the SDK and set your Fusion AppId |
| [Adrenak UniVoice / UniMic](https://github.com/adrenak/univoice) | Referenced by the runtime assembly definition |
| TextMeshPro 3.0.6 and XR Core Utils 2.2.3 | Installed automatically by UPM |

## Installation

**Unity Package Manager (Git URL)**

1. `Window > Package Manager > + > Add package from git URL...`
2. Enter `https://github.com/TurboT22/FusionVR.git`

**From a zip / local folder**

1. Unzip the package.
2. `Window > Package Manager > + > Add package from disk...` and pick `package.json`.

## Installing UniVoice

**Automatic:** `Tools > FusionVR > Install UniVoice` (or the Install button in the Setup Wizard) edits
`Packages/manifest.json` for you and keeps a `.fusionvr.bak` backup.

**Manual:**

UniVoice is distributed through the **npmjs registry** (OpenUPM may be out of date).

1. `Edit > Project Settings > Package Manager`, then add a **Scoped Registry**:
   - Name: `npmjs`
   - URL: `https://registry.npmjs.org`
   - Scopes: `com.adrenak.univoice`, `com.adrenak.brw`, `com.adrenak.unimic`, `com.adrenak.concentus-unity`
2. `Window > Package Manager > My Registries`, find **UniVoice**, and click Install. Its dependencies (UniMic,
   Concentus, BRW) install with it.

## Quick Start (1-click Setup Wizard)

1. Open `Tools > FusionVR > Setup Wizard`.
2. Make sure Photon Fusion 2, UniVoice and TextMeshPro Essentials show green ticks.
3. Enter your **Fusion AppId** and **Region**, then click **Apply**.
4. For Meta Quest, click **Patch Manifest** to add the `RECORD_AUDIO` permission.
5. Open your VR scene and click **Setup Active Scene Automatically**. This generates the prefabs, adds
   `FusionVRManager`, detects your XR rig (`XROrigin` or `OVRCameraRig`) and wires everything.
6. Press Play. With **Connect On Awake** and **Join Room On Connect** enabled, you join the default queue.

If `PlayerPrefab` stays empty, run `Tools > Fusion > Rebuild Prefab Table`, then
`Tools > FusionVR > Rebuild & Re-register Prefabs`.

## Usage

```csharp
using Fusion.VR;
using UnityEngine;

public class Example : MonoBehaviour
{
    void Start()
    {
        FusionVRManager.OnJoinedRoom += () => Debug.Log("Joined " + FusionVRManager.CurrentRoomName);
        FusionVRManager.OnError += msg => Debug.LogWarning(msg);

        FusionVRManager.SetUsername("Alice");
        FusionVRManager.SetColour(Color.cyan);
    }
}
```

### Connecting

```csharp
FusionVRManager.Connect();                                      // AppId + Region from the manager
FusionVRManager.ChangeServers("new-app-id");                    // switch AppId and reconnect
FusionVRManager.ChangeServers("new-app-id", "us");              // switch AppId and region
FusionVRManager.ConnectAuthenticated("username", "token");      // custom auth (PlayFab etc.)
FusionVRManager.ChangeServersAuthenticated("app-id", "user", "token");
```

### Rooms

```csharp
FusionVRManager.JoinRandomRoom("Default", 10);   // public room matched by queue
FusionVRManager.JoinPrivateRoom("ABC123", 8);    // private room by code (upper-cased)
FusionVRManager.SwitchScenes(1, 8);              // load scene 1 and join its own room
FusionVRManager.LeaveRoom();
```

### Appearance and cosmetics

```csharp
FusionVRManager.SetUsername("Bob");
FusionVRManager.SetColour(new Color(1f, 0.4f, 0.1f));

FusionVRManager.SetCosmetic("Head", "VRTopHat");
FusionVRManager.SetCosmetics(new Dictionary<string, string>
{
    { "Head", "VRTopHat" },
    { "Face", "VRSunglasses" }
});
FusionVRManager.SetCosmetic("Head", "");   // remove
```

Add cosmetics with `Tools > FusionVR > Cosmetic Helper`, or by hand: put a mesh under
`FusionVRPlayer/Head/Cosmetics/` and name the GameObject with your cosmetic ID.

### Voice

```csharp
FusionVRManager.SetMuted(true);
FusionVRManager.ToggleMute();
FusionVRManager.SetDeafened(true);
FusionVRManager.SetPlayerVolume(somePlayerRef, 0.5f);

var mics = FusionUniVoiceTransport.Instance.GetAvailableMicrophones();
FusionUniVoiceTransport.Instance.SetMicrophoneDevice(mics[0]);
```

### Events

```csharp
FusionVRManager.OnConnected        += () => { };
FusionVRManager.OnJoinedRoom       += () => { };
FusionVRManager.OnLeftRoom         += () => { };
FusionVRManager.OnPlayerJoinedRoom += player => { };
FusionVRManager.OnPlayerLeftRoom   += player => { };
FusionVRManager.OnError            += message => { };
```

### Players

```csharp
foreach (var pair in FusionVRManager.Players)
    Debug.Log(pair.Value.Username + " " + pair.Value.Colour);
```

## World helpers

- **`FusionVRInteractableButton`**: trigger collider pressed by a hand (tag and/or layer). Plays a press offset,
  click sound and haptics, raises `OnButtonPressed`, and offers quick actions (join queue, join private room,
  switch scene, colour, cosmetic, mute, deafen).
- **`FusionVRRoomKeypad`**: assign `RoomCodeText` and `StatusText`. Child buttons named `Key_0`-`Key_9`,
  `Key_A`-`Key_Z`, `Key_Backspace`, `Key_Clear` and `Key_Enter` are bound automatically.
- **`FusionVRNameplateBillboard`**: faces the camera and hides on your own avatar.

## How voice works

The microphone is captured at `SampleRate`, gated by voice activity, mu-law compressed to 8 bit, and sent to nearby
players with `SendReliableDataToPlayer` using the `FVR1` key. Packets are split into segments under 900 bytes and
reassembled on arrival. Each remote avatar's `FusionVRVoiceSpeaker` plays the audio as 3D spatial sound and exposes
`VoiceLoudness` (0-1) and `IsSpeaking` for lip-sync. Players beyond `MaxVoiceDistance` are silent and are not sent audio.

## Troubleshooting

| Problem | Fix |
|---|---|
| `PlayerPrefab is not set` | Run the Setup Wizard, or Rebuild & Re-register Prefabs |
| Spawn error / unknown prefab | `Tools > Fusion > Rebuild Prefab Table` |
| No voice on Quest | Patch the manifest in the wizard and accept the microphone prompt |
| Can't connect | Check the Fusion AppId (not the Realtime or Voice AppId) |
| Scene change loses rig | Keep a `FusionVRManager` in each scene; the persistent one adopts the new rig references |

## License

MIT. See [LICENSE](LICENSE).

# MindRun Windows Build

## Citation

If you use this software in academic work, demonstrations, experiments, or derivative research, please cite the associated publication once it becomes available.

The paper describing MindRun is expected for the Graz BCI Conference, but it has not been published yet. Until the final proceedings citation is available, please cite this build/repository and mention the forthcoming Graz conference paper:

Palatella, A., Tortora, S., Menegatti, E., Tonin, L., and Alimardani, M. MindRun: an Accessible Multi-Player BCI Game for Collaborative Motor Imagery Training.

Suggested temporary citation:

```bibtex
@inproceedings{palatella_mindrun_2026,
  author = {Palatella, Alessio and Tortora, Stefano and Menegatti, Emanuele and Tonin, Luca and Alimardani, Maryam},
  title = {MindRun: an Accessible Multi-Player BCI Game for Collaborative Motor Imagery Training},
  booktitle = {Proceedings of the Graz BCI Conference},
  year = {2026},
  note = {Forthcoming}
}
```

## Overview

MindRun is a Windows build of a Unity-based multi-player BCI game for collaborative motor imagery training.

The build can be used with the accompanying g.tec EEG pipeline to receive real-time BCI control values, or with another BCI system that provides the expected online control signal.

Related EEG pipeline:

```text
https://github.com/A13ssi0/gTec_EEGpipeline
```

## Included Files

```text
.
|-- bci_game.exe                 # Main Windows executable
|-- bci_game_Data/               # Unity data folder required by the executable
|-- D3D12/                       # Unity/Direct3D runtime files
|-- MonoBleedingEdge/            # Unity Mono runtime files
|-- UnityPlayer.dll              # Unity player library
|-- UnityCrashHandler64.exe      # Unity crash handler
|-- LICENSE
`-- README.md
```

## Requirements

- Windows 64-bit.
- The complete Unity build folder, not only `bci_game.exe`.
- For BCI control: the EEG pipeline must be running and configured to stream the control output expected by the game.

Keep `bci_game.exe` in the same folder as `bci_game_Data/`, `UnityPlayer.dll`, `D3D12/`, and `MonoBleedingEdge/`. Moving only the `.exe` will prevent the game from starting.

## Running The Game

1. Extract or clone the full build folder.
2. If using BCI control, start the EEG pipeline first.
3. Run:

```text
bci_game.exe
```

4. If Windows SmartScreen warns about an unknown publisher, choose the option to run the application only if you trust the source.

## BCI Pipeline Use

For online BCI experiments, run the EEG pipeline in evaluation or test mode before launching the game. The pipeline should provide the real-time control value produced by the classifier/output mapper.

Typical setup:

1. Start the EEG pipeline.
2. Confirm acquisition, filtering, classification, and output mapping are running.
3. Start `bci_game.exe`.
4. Run the experiment/game session.

For hardware-free testing, use the test configuration of the EEG pipeline with the included test model.

## External BCI Integration

MindRun does not require the provided EEG pipeline specifically. Other BCI systems can control the game if they provide the same kind of online control signal expected by the build.

The game control value is a normalized horizontal position/control value, commonly called `percPosX` in the accompanying pipeline:

```text
0.0   left side
0.5   center / neutral
1.0   right side
```

Values should be sent continuously during gameplay and clipped to the range `[0, 1]`.

When using `gTec_EEGpipeline`, this value is produced by the `OutputMapper` node after classifier probabilities are integrated. In the Python pipeline, `percPosX` is sent as a TCP payload containing the ASCII representation of a float, for example:

```text
0.5
0.613
0.982
```

The Python pipeline wraps TCP payloads with a small timestamped framing protocol:

```text
4 bytes  timestamp length, big-endian integer
N bytes  timestamp string, formatted as HH:MM:SS.ffffff
4 bytes  payload length, big-endian integer
M bytes  payload
```

An external BCI program can therefore connect to the game using the same TCP mode, or use the direct TCP/UDP connection mode configured in the Unity build, and stream a normalized control value at the desired update rate.

The recommended integration behavior is:

- Send one floating-point control value at a regular rate.
- Keep the value between `0.0` and `1.0`.
- Use `0.5` as the neutral/no-control state.
- If your classifier rejects a window or has low confidence, send `0.5`.
- Start the control server/client before beginning the game session so MindRun can connect cleanly.

If event synchronization is needed, use the event channel expected by your experiment setup. In the accompanying EEG pipeline, game events are sent as event-code messages so that EEG recordings can be aligned with game/trial events.

## License

This build is not released under an open-source license. See `LICENSE` for the usage and redistribution terms.

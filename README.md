# DigiPhant Unity project

Open `Assets/DigiPhant/Scenes/DigiPhant.unity` in Unity **6000.6.10f1**.
See [DIGIPHANT_SETUP.md](DIGIPHANT_SETUP.md) for setup, live testing and recording.

## Restore a fresh checkout

From the student project root on macOS, with Python 3.14 installed:

```sh
git clone https://github.com/kommanderpi/studentstarter.git DigiPhantStarter
git -C DigiPhantStarter checkout --detach 4e6dab5271c618d1c0bfb2022c41c0d37cd8d69f
cp -R DigiPhantStarter/Elephant Assets/Elephant
cp DigiPhantStarter/Elephant.meta Assets/Elephant.meta
python3.14 -m venv DigiPhantStarter/Tracking/.venv
DigiPhantStarter/Tracking/.venv/bin/python -m pip install -r DigiPhantStarter/Tracking/requirements.txt
DigiPhantStarter/Tracking/.venv/bin/python -m pip check
```

Run the copy commands on a clean checkout before opening Unity. Preserve the
tracked `Assets/DigiPhant` folder. The pose model and Python scripts come from
the pinned clone under `DigiPhantStarter/Tracking`. There are no tracking patches
or modified elephant assets to restore.

## Project and performance links

- [Student project repository](https://github.com/ethanswhong/digiphant_test)
- Drive performance video: https://drive.google.com/file/d/1okuIqFErf4iw3BogBS_06MYVXssV7z83/view?usp=sharing

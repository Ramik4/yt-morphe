# Morphe Patches test builds

Release YT Morphe with the `morphe` input (such as `RAMiK4/morphe-patches@<branch>`) builds only the Amoled variant
(`app.morphe.android.youtube`) from Morphe Patches, with its default patches. No RLX code, so the RLX default
settings (`rlx/defaults.json` of RLX Patches) are not applied.

`settings.json` holds the RLX defaults that also exist in Morphe, for Settings → Other → Import / export → Import
on a fresh install. Left out: the RLX logo and settings name, the RLX theme colors (Amoled uses pure black and
white), Yandex voice over translation and the settings Morphe does not have.

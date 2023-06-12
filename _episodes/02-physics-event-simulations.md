---
title: "Running physics simulations with `ddsim`"
teaching: 30
exercises: 20
questions:
- "How can I simulate events from physics event generators?"
objectives:
- "`ddsim` can simulate HepMC3 events in DD4hep geometries."
- "`npsim` can be used as an alternative for simulations with optical photons."
keypoints:
- "`ddsim` or `npsim` are both able to simulate physics events."
---
We now move on to running simulations on HepMC3 event input files from (in this case) Pythia8. The large input files for simulation campaigns are stored on S3, but the `eic-shell` environment contains the Minio cient `mc` to access them. We first need to set up our access credentials, though:
```console
mc config host add S3 https://eics3.sdcc.bnl.gov:9000 $S3_ACCESS_KEY $S3_SECRET_KEY
mc ls S3/eictest/EPIC/Tutorials/pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.hepmc
```

> Note: The values for `$S3_ACCESS_KEY` and `$S3_SECRET_KEY` will be provided in the tutorial, but are not provided here.
{: .callout}

You will notice that the input file is large (GBs). For this tutorial we only need the first few thousand lines:
```console
mc head -n 20000 S3/eictest/EPIC/Tutorials/pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.hepmc > pythia8NCDIS_10x100.hepmc
```

We can now specify this HepMC3 input file as input to `ddsim`:
```console
ddsim --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml --numberOfEvents 10 --inputFiles pythia8NCDIS_10x100.hepmc --outputFile pythia8NCDIS_10x100.edm4hep.root
```

# NPSim as an alternative to `ddsim`

Since some of the options that we pass to `ddsim` can only be provided through a steering file (such as python functions), or are otherwise cumbersome to provide on the command line, we provide `npsim` as a layer on top of `ddsim` that has these options pre-configured. This is as if you would take your steering file options and contribute them back to a central location for others to use them.

`npsim` can be easily interpreted (since it has sections that look exactly like the steering file). We can look at its python source code, located at `/usr/local/bin/npsim.py` in the `eic-shell` environment.

Currently `npsim` has the following additional options:
- Cerenkov and optical photon physics are added through a python setup function,
- an optical photon filter is created and added to the DRICH,
- a modified tracking detector action which absorbs optical photon,
- the minimal energy deposition in the tracker is set to zero.

You can run `npsim` exactly as you would run `ddsim`.

> Exercise:
> - Rerun the previous Pythia8 simulation with `npsim`, and notice any difference in running time. Because of the addition of optical photon physics, the simulation will run more slowly.
> - Open the output file and verify that more hits (from optical photons with PDG code -22) are stored in the hits branches for the relevant RICH detector.
{: .challenge}


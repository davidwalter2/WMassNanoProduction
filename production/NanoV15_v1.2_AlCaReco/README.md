# NanoV15_v1.2 AlCaReco production: J/psi and Upsilon, 2016 postVFP (runbook)

Tracking: one issue for the whole production, opened when the code is merged. Coordination and submission:
@davidwalter2. The general steps (proxy, monitoring, exit codes) are those of the
[NanoV15_v1.0 runbook](../NanoV15_v1.0/README.md); this page has what is specific to the AlCaReco nano.

## 0. What is produced

The AlCaReco nano (`PhysicsTools/NanoAOD/python/alcareco_nano_cff.py`) refits the muon tracks of the
tracker-alignment AlCaReco skims with the CVH single-track refit of the custom NanoAOD. It has two versions:

- **default**: the Muon table with the CVH columns and the Jacobians from which the module-level corrections are
  *applied*, as in the MiniAOD NanoAOD;
- **calibration** (`--calib`, J/psi only): additionally the content from which they are *derived*:
  - the two-track refit of the J/psi candidates with the common-vertex and the J/psi mass constraint;
  - its Dimuon table with the global-fit payload (gradient, factored Hessian, indices);
  - the parameter catalog in the Runs tree.

  It is read by the nano mode of `Analysis/aggregategrads.py` in MuonMomentumScaleCalibration
  (davidwalter2/MuonMomentumScaleCalibration `main`).

**Six productions:** J/psi data and MC in both versions, Upsilon data and MC in the default version.

- **Code:** tag `NanoV15_v1.2` in both repositories (WMass/cmssw `WmassNanoProd_15_0_19_patch2`, WMass/WMassNanoProduction
  `main`). The J/psi calibration version was submitted before the merge, from the heads of WMass/cmssw#51
  (`8af6576483f`) and WMass/WMassNanoProduction#31 (`679c653`), which are the code of the tag: #51 is not behind
  its base, and the base commits #31 does not have change only history files and the v1.0 runbook.
- **Inputs:** `inputs/alcareco_data_postVFP.txt` and `inputs/alcareco_mc_postVFP.txt`, the samples of AN-21-131. Events
  and replicas are in `inputs/INVENTORY_ALCARECO_261016.md`.
- **Output:** site `T3_US_MIT`, LFN `/store/group/zmass/NanoAOD`, i.e.
  `/ceph/submit/data/group/cms/store/group/zmass/NanoAOD/<primary dataset>/<output tag>/<YYMMDD_HHMMSS>/<NNNN>/*.root`.
  - Output tags `NanoV15<skim>[Calib]Run<era>DataPostVFP_v1p2` and `NanoV15<skim>[Calib]MCPostVFP_v1p2`, e.g.
    `NanoV15TkAlJpsiMuMuCalibRun2016HDataPostVFP_v1p2`.
  - CRAB label `-v v1p2`.
- **Jobs:** `EventAwareLumiBased`, 8 CPU hours each. The calibration version runs with 2 threads (`-j 2`: 2 cores,
  4000 MB, about 4.5 h), the default version with 4 (8000 MB, about 2.2 h); the peak memory is 1.9–2.2 GB.

| version | data [s CPU/event] | MC [s CPU/event] | events/job (data / MC) | output [kB/event] (data / MC) |
|---|---|---|---|---|
| default | 0.09 | 0.19 | 320k / 152k | 2.1 / 4.4 |
| calibration | 0.34 | 0.46 | 85k / 63k | 37 / 43 |

The calibration version takes 3.8x the CPU of the default version in data and 2.41x in MC, measured on submit82
on the same events (2281 J/psi 2016H events: 0.137 vs 0.036 s/event; 2000 UL16 J/psi Pt8toInf events: 0.184 vs
0.076 s/event), and is scaled to the grid with the default version's numbers (README, "Job size"). The output
sizes are those of the Events trees of the test files. A calibration output file is about 3 GB.

The calibration version does not use 4 threads well. On the same 3000 J/psi 2016H events (read in order from a
local file, on submit82) its event loop used 0.84 cores with 1 thread, 1.62 with 2 and 2.19 with 4, at 0.137,
0.139 and 0.148 s CPU/event; with 2 threads a core does 1.6x the work it does with 4. The output does not
depend on the number of threads.

| dataset | events (DAS) | on disk at | jobs, default | jobs, calibration |
|---|---|---|---|---|
| `/Charmonium/Run2016F-TkAlJpsiMuMu-21Feb2020_UL2016-v1/ALCARECO` | 2,288,895 | T2_CH_CERN | 8 | 27 |
| `/Charmonium/Run2016G-TkAlJpsiMuMu-21Feb2020_UL2016-v1/ALCARECO` | 41,103,908 | T2_CH_CERN | 129 | 484 |
| `/Charmonium/Run2016H-TkAlJpsiMuMu-21Feb2020_UL2016-v1/ALCARECO` | 46,975,701 | T2_CH_CERN | 147 | 553 |
| `/JPsiToMuMu_Pt0to8-pythia8/RunIISummer20UL16RECO-TkAlJpsiMuMu-106X_mcRun2_asymptotic_v13-v2/ALCARECO` | 456,470 | T2_CH_CERN | 4 | 8 |
| `/JPsiToMuMu_Pt0to8-pythia8/RunIISummer20UL16RECO-TkAlJpsiMuMu-106X_mcRun2_asymptotic_v13_ext1-v3/ALCARECO` | 1,373,184 | T2_CH_CERN | 10 | 22 |
| `/JPsiToMuMu_Pt8toInf-pythia8/RunIISummer20UL16RECO-TkAlJpsiMuMu-106X_mcRun2_asymptotic_v13-v2/ALCARECO` | 30,117,032 | T2_CH_CERN | 199 | 479 |
| `/JPsiToMuMu_Pt8toInf-pythia8/RunIISummer20UL16RECO-TkAlJpsiMuMu-106X_mcRun2_asymptotic_v13_ext1-v3/ALCARECO` | 90,839,033 | T2_CH_CERN | 598 | 1442 |
| `/MuOnia/Run2016F-TkAlUpsilonMuMu-31Aug2023_UL2016_WMass-v1/ALCARECO` | 214,784 | T2_US_MIT (recall) | 1 | – |
| `/MuOnia/Run2016G-TkAlUpsilonMuMu-31Aug2023_UL2016_WMass-v1/ALCARECO` | 3,782,522 | T2_US_MIT (recall) | 12 | – |
| `/MuOnia/Run2016H-TkAlUpsilonMuMu-31Aug2023_UL2016_WMass-v1/ALCARECO` | 4,235,035 | T2_US_MIT (recall) | 14 | – |
| `/Upsilon_MultiResonanceFilter_Pt0to8-pythia8/RunIISummer20UL16RECO-TkAlUpsilonMuMu-106X_mcRun2_asymptotic_v13-v2/ALCARECO` | 2,980,640 | T2_US_MIT (recall) | 20 | – |
| `/Upsilon_MultiResonanceFilter_Pt8to13-pythia8/RunIISummer20UL16RECO-TkAlUpsilonMuMu-106X_mcRun2_asymptotic_v13-v2/ALCARECO` | 13,756,522 | T2_US_MIT (recall) | 91 | – |
| `/Upsilon_MultiResonanceFilter_Pt13toInf-pythia8/RunIISummer20UL16RECO-TkAlUpsilonMuMu-106X_mcRun2_asymptotic_v13-v2/ALCARECO` | 47,961,760 | T2_US_MIT (recall) | 316 | – |

| production | events | jobs | CPU hours | output |
|---|---|---|---|---|
| J/psi data, default | 90.4M | 284 | 2,300 | 0.2 TB |
| J/psi data, calibration | 90.4M | 1,064 | 8,500 | 3.3 TB |
| J/psi MC, default | 122.8M | 811 | 6,500 | 0.5 TB |
| J/psi MC, calibration | 122.8M | 1,951 | 15,700 | 5.3 TB |
| Upsilon data, default | 8.2M | 27 | 200 | 0.02 TB |
| Upsilon MC, default | 64.7M | 427 | 3,400 | 0.3 TB |
| **total** | | **4,564** | **36,600** | **9.7 TB** |

## 1. Prerequisites

- **Account:** as in step 1 of the v1.0 runbook (submit-cms group, grid proxy, `git config --global user.github`).
- **Code:** WMass/cmssw#51, WMass/WMassNanoProduction#31 and #34 are merged, and `NanoV15_v1.2` is tagged in both
  repositories.
- **Upsilon inputs on disk:** MuOnia and the Upsilon MC (2.6 TB) are on tape. They are recalled to T2_US_MIT with one
  Rucio rule per dataset, in the `User AutoApprove` activity, which needs no site approval for a lifetime of at most
  30 days:
  ```sh
  source /cvmfs/cms.cern.ch/rucio/setup-py3.sh && export RUCIO_ACCOUNT=dwalter
  for ds in $(grep -hE '^/(MuOnia|Upsilon)' inputs/alcareco_data_postVFP.txt inputs/alcareco_mc_postVFP.txt); do
    rucio list-rules cms:$ds | tail -n +3
    rucio add-rule --ask-approval --activity "User AutoApprove" --lifetime 2592000 \
      --comment "WMass AlCaReco nano, Upsilon" cms:$ds 1 T2_US_MIT
  done
  ```
  The rules were created on 2026-10-22, were approved automatically, and expire on 2026-11-21:

  | dataset | rule |
  |---|---|
  | MuOnia Run2016F | `7f0563e2782a4d3983f768cf3a85bd52` |
  | MuOnia Run2016G | `44416e6fe66649b285a7c797e56b696c` |
  | MuOnia Run2016H | `5bb1153d2ff243f2b7780b7bc1bff145` |
  | Upsilon_MultiResonanceFilter Pt0to8 | `60749562c7bb45338a431fc7fd40647c` |
  | Upsilon_MultiResonanceFilter Pt8to13 | `042cdd50d2944b4f804fd9719acc0d23` |
  | Upsilon_MultiResonanceFilter Pt13toInf | `46c4e403ce3a4aca8c320cf50a544700` |

  - `rucio list-rules --account dwalter` shows the rules `REPLICATING`, then `OK`; `WAITING_APPROVAL` means the
    request fell outside the automatic approval;
  - submit the Upsilon tasks when the rules are `OK`, and extend the lifetime
    (`rucio update-rule --lifetime <seconds> <rule id>`) if the production runs longer.
- **J/psi inputs:** all valid files are on disk at T2_CH_CERN. DAS shows Charmonium Run2016H at 97.6 % block
  completion because one of its 42 blocks is empty (0 files); all 122 files are at T2_CH_CERN.

## 2. Setup at the tag

```sh
mkdir -p /work/submit/$USER/NanoV15_v1.2 && cd /work/submit/$USER/NanoV15_v1.2
CMSSW_TOPIC=WMass:NanoV15_v1.2 PROD_BRANCH=NanoV15_v1.2 PROD_REPO=https://github.com/WMass/WMassNanoProduction.git \
  bash <(curl -s https://raw.githubusercontent.com/WMass/WMassNanoProduction/NanoV15_v1.2/setup/clone_15_0.sh)
cd CMSSW_15_0_19_patch2/src && cmsenv && cd Configuration/WMassNanoProduction
git describe --tags        # NanoV15_v1.2
./scripts/prepareSandbox.sh
source /cvmfs/cms.cern.ch/common/crab-setup.sh
crab checkwrite --site=T3_US_MIT --lfn=/store/group/zmass/NanoAOD/
```

`prepareSandbox.sh` must report the OPERA tables with `sha1 ok`; `crab checkwrite` must end with `Success`.

## 3. Submit

```sh
OPTS="-v v1p2 --site T3_US_MIT --storage /store/group/zmass/NanoAOD"
```

Each production is two commands: the first writes the configs and the submit files and prints, per dataset, the
events per job and the number of jobs; the second submits. Between the two, check the submit files:

```sh
grep -h "numCores\|maxMemoryMB\|inputDataset\|splitting\|unitsPerJob\|outLFNDirBase\|outputDatasetTag\|storageSite" crab_submit/submit*.py
```

They must show 2 cores and 4000 MB (calibration version) or 4 cores and 8000 MB (default version),
`EventAwareLumiBased`, the events per job of the first table in step 0,
`/store/group/zmass/NanoAOD`, the output tag (with `Calib` for the calibration version) and `T3_US_MIT`.

| production | list | options |
|---|---|---|
| J/psi data, default | `inputs/alcareco_data_postVFP.txt` | `-m '/Charmonium'` |
| J/psi data, calibration | `inputs/alcareco_data_postVFP.txt` | `-m '/Charmonium' --calib -j 2` |
| J/psi MC, default | `inputs/alcareco_mc_postVFP.txt` | `-m '/JPsiToMuMu'` |
| J/psi MC, calibration | `inputs/alcareco_mc_postVFP.txt` | `-m '/JPsiToMuMu' --calib -j 2` |
| Upsilon data, default (after the recall) | `inputs/alcareco_data_postVFP.txt` | `-m '/MuOnia'` |
| Upsilon MC, default (after the recall) | `inputs/alcareco_mc_postVFP.txt` | `-m '/Upsilon'` |

```sh
./scripts/prepareCrab.py --makeConfig -i <list> $OPTS <options>
./scripts/prepareCrab.py -i <list> $OPTS <options> --submit 1 1
```

`--calib` is refused for the Upsilon inputs. The CRAB project directories are `crab_submit/crab_projects/crab_<task>`,
and every submission is appended to `history/<list>_<date>_submit11_<user>.txt`.

## 4. Monitor

As step 4 of the v1.0 runbook: `crab status -d <dir>` once or twice a day, the exit-code table there, and the
task's Grafana page (the `Dashboard monitoring URL` line of `crab status`).

- All J/psi inputs are on disk at T2_CH_CERN only, so the jobs may wait there.
- The run-boundary crash (exit code 8901 in data, v1.0 runbook) is fixed since `NanoV15_v1.1`. An 8901 is treated
  like the other codes: resubmit, and report it in the issue if it repeats.

## 5. Recover jobs that keep failing

`crab recover` does not support `EventAwareLumiBased` tasks, so recover by hand, once the remaining jobs have failed
their retries. The recovery task keeps the job size of the original one. In `crab_submit/`:

```sh
crab kill -d crab_projects/crab_<task>
crab report -d crab_projects/crab_<task>      # writes crab_projects/crab_<task>/results/notFinishedLumis.json
cp submit<name>.py submit<name>_rec1.py
```

In `submit<name>_rec1.py`, change only:

```python
config.General.requestName = '<original requestName>_rec1'
config.Data.lumiMask = 'crab_projects/crab_<task>/results/notFinishedLumis.json'
```

Then `crab submit -c submit<name>_rec1.py`. It writes to the same output tag (a new timestamp directory).

## 6. Completion checks (per task)

1. `crab report -d <dir>`: the events read must equal the DAS events of step 0; for a recovered task, add up the task
   and its `_rec` tasks.
2. `ls -ld /ceph/submit/data/group/cms/store/group/zmass/NanoAOD/<PD>/<output tag>` shows `drwxr-sr-x`.
3. Calibration version: every run of every file has the parameter catalog. With `cmsenv`, for
   `<output dir>` = `/ceph/submit/data/group/cms/store/group/zmass/NanoAOD/<PD>/<output tag>`:
   ```sh
   python3 -c "import ROOT, glob, sys; fs = glob.glob(sys.argv[1]); n = ROOT.RDataFrame('Runs', fs).AsNumpy(['nCvhParm'])['nCvhParm']; print(len(fs), 'files', len(n), 'runs, nCvhParm', n.min(), '-', n.max())" '<output dir>/*/*/*.root'
   ```
   It must print `nCvhParm 116972 - 116972`.
4. Calibration version, data: `aggregategrads.py` reads the files. On submit82, as it needs about 55 GB of memory
   (the Hessian of 116972 parameters), in an empty directory:
   ```sh
   ROOT_MAX_THREADS=16 CVH_GRADS_INPUTS='<output dir>/*/*/*.root' \
     apptainer exec -B /tmp,/home/submit,/work/submit,/scratch/submit,/ceph/submit,/cvmfs,/etc/grid-security,/run \
     /cvmfs/unpacked.cern.ch/gitlab-registry.cern.ch/bendavid/cmswmassdocker/wmassdevrolling:latest \
     python3 <MuonMomentumScaleCalibration>/Analysis/aggregategrads.py 5
   ```
   `<MuonMomentumScaleCalibration>` is a clone of davidwalter2/MuonMomentumScaleCalibration with its `narf` submodule.
   The argument limits it to 5 files. It must print `input format: calibration NanoAOD` and
   `parameter catalog: 116972 parameters`, and it writes `combinedgrads.hdf5` and `runtree.root`. It applies the 2016
   golden JSON, so it selects nothing in MC.
5. Commit the history files on a branch (`history-NanoV15_v1.2_AlCaReco-$USER`) and open a PR to
   `WMass/WMassNanoProduction` `main`.
6. Post the completion report in the issue, in the format of step 7 of the v1.0 runbook.

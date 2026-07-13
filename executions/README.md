# executions/ — auto-generated

RobNTex writes every run's output under one timestamped folder here:

```
executions/execu_<YYYY-MM-DD_HH-MM-SS>/
├── report.md                           # per-TC summary + tally
├── testcases/                          # copy of the TCs that ran (traceability)
├── browser-sessions/
│   └── <session>/
│       ├── logs/                       # console + network per TC
│       └── screenshots/
│           └── <TC-ID>/                # one folder per TC
│               ├── step-1.png
│               ├── step-2.png
│               └── ...
└── bugs/
    ├── bug-list.md                     # Jira-ready — one block per defect
    └── screenshots/                    # copies of bug-evidence shots
```

Nothing is created here until a run happens. This folder is `.gitignore`d — commit the
report only if you explicitly want it in version control.

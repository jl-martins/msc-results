# MSc Results

Energy, execution time and hot loop analysis results from my Master's Dissertation on the Impact of Code Rejuvenation on Energy Consumption.

The measurements and analyses cover the loop-based (`loops`) and stream-based (`streams`) implementations of 15 benchmarks from [msc-dacapobench](https://github.com/jl-martins/msc-dacapobench), a research fork of [DaCapo Chopin](https://doi.org/10.1145/3669940.3707217) that I created for the dissertation. They were produced with [msc-research-tools](https://github.com/jl-martins/msc-research-tools), a set of tools that I developed to measure, profile and analyze these benchmarks.

## Layout

```text
├── energy-and-time/   # energy and execution time measurements (dissertation, chapter 5)
└── hot-loops/         # hot loop analysis (dissertation, section A.1)
```

### Energy and Time

`energy-and-time` holds the measurements from `jbench-run` and the statistics from `bench-stats`. Each benchmark ran with the G1 collector and a fixed heap of twice its overall cold minimum heap size (dissertation, section 4.7).

```text
energy-and-time/
├── energy_summary.csv                    # effect of converting loops to sequential streams on energy, by JVM state
├── energy_summary_by_jdk.csv             # same, by JVM state and JDK
├── energy_time_correlation.csv           # correlation between energy and time changes
└── <state>/                              # cold (-n 1) or warm (-n 5)
    ├── cpuinfo.txt, raminfo.txt, …       # system configuration
    ├── streams_*.pgf                     # cross-JDK plots
    ├── streams_absolute_energy_time.csv  # medians and IQRs of energy and time
    └── <jdk>/                            # temurin-8, -11, -17, -21 or -25
        ├── java_version.txt              # output of java -version
        ├── loops/                        # loop-based implementation
        │   └── <benchmark>/              # measurements of one benchmark
        └── streams/                      # stream-based implementation
            ├── <benchmark>/              # measurements of one benchmark
            ├── comparison.csv            # effect sizes of streams relative to loops
            └── *_forest_plot.pgf         # energy effect sizes and their CIs, by benchmark
```

Each `temurin-<N>` directory holds the results of Eclipse Temurin `<N>`, a build of Java `<N>`. `java_version.txt` gives the exact release.

Not every benchmark is compatible with every Java version (dissertation, Table 3). Temurin 8 runs 9 benchmarks, Temurin 25 runs 14 (all but cassandra), and the other JDKs run all 15.

Each `<benchmark>/` directory holds three files:

* `measurements.csv` - one row per JVM invocation (20 per benchmark)
* `java_options.txt` - JVM options of every invocation
* `suite_options.txt` - DaCapo harness options of every invocation

The columns of `measurements.csv` come from `RAPLCallback`, a DaCapo callback that I implemented in msc-dacapobench to measure the energy consumption of DaCapo benchmarks. Every column refers to the last benchmark iteration:

| Column                              | Unit | Description                                       |
| ----------------------------------- | ---- | ------------------------------------------------- |
| `Package`, `Core`, `Uncore`, `DRAM` | J    | Energy consumption of each Intel RAPL domain      |
| `Time`                              | ms   | Execution time                                    |
| `Temperature`                       | °C   | CPU package temperature after the iteration       |
| `Memory`                            | KiB  | Peak resident set size                            |
| `Powercap`                          | -    | Power cap (`-1` means no power cap)               |

### Hot Loop Analysis

`hot-loops` holds the outputs of `dacapo-hot-loops`.

```text
hot-loops/
├── study.toml                        # configuration of the study
├── profiles/                         # output of the `profile` subcommand
│   └── <benchmark>/
│       ├── <benchmark>-profile.txt   # async-profiler profile in collapsed format, with line numbers
│       └── <benchmark>-profile.html  # same profile, as a flame graph
├── analysis/                         # output of the `analyse` subcommand: hot methods, chargeable loops and hot loops
├── classification.csv                # my classification of the 90 sampled hot loops
├── report/                           # output of the `report` subcommand: share of hot loops suitable for streams
└── coverage/                         # output of the `coverage` subcommand: hot loops converted to streams
```

`analysis/`, `report/` and `coverage/` each have a `provenance.json` file with the SHA-256 digests of their configuration and inputs.

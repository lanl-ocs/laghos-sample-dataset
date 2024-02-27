**A sample laghos 3D mesh dataset for local OCS development and testing**

[![License](https://licensebuttons.net/l/by/4.0/88x31.png)](https://creativecommons.org/licenses/by/4.0/)

OCS Laghos Sample Dataset
=

```
XX              XXXXX XXX         XX XX           XX       XX XX XXX         XXX
XX             XXX XX XXXX        XX XX           XX       XX XX    XX     XX   XX
XX            XX   XX XX XX       XX XX           XX       XX XX      XX XX       XX
XX           XX    XX XX  XX      XX XX           XX       XX XX      XX XX       XX
XX          XX     XX XX   XX     XX XX           XX XXXXX XX XX      XX XX       XX
XX         XX      XX XX    XX    XX XX           XX       XX XX     XX  XX
XX        XX       XX XX     XX   XX XX           XX       XX XX    XX   XX
XX       XX XX XX XXX XX      XX  XX XX           XX XXXXX XX XX XXX     XX       XX
XX      XX         XX XX       XX XX XX           XX       XX XX         XX       XX
XX     XX          XX XX        X XX XX           XX       XX XX         XX       XX
XX    XX           XX XX          XX XX           XX       XX XX          XX     XX
XXXX XX            XX XX          XX XXXXXXXXXX   XX       XX XX            XXXXXX
```

This sample dataset is generated by running the open source Laghos miniapp using a single MPI process on a single node with result converted into apache parquet (from mfem). 

**There are 256 equally-formatted parquet files in this dataset.** Each stores the state of a specific simulation timestep, from timestep 36000 to timestep 37275. The simulation had 524,287 mesh elements and 612,310 vertices, resulting in 4,194,304 nodal values per timestep, and 4,194,304 rows per parquet file.

**Each of these parquet files consists of a single row group and 10 columns**: `element_id`, `vertex_id`, `v_x`, `v_y`, `v_z`, `rho`, `e`, `x`, `y`, and `z`. Each column represents an attribute of a nodal value, and is formatted as an array of snappy-compressed, plain-encoded parquet pages. There are 16-32 pages per column per row group depending on data types (whether it's 4 or 8 bytes). Applying snappy compression on all columns reduces the size of each parquet file from 288MB to 97MB, leading to more efficient storage.

Before uploading to github, these parquet files are additionally compressed using brotli. This further reduces the parquet file size from 97MB to 39MB so that github does not complain file being too large.

To recover a parquet file from its Brotli compressed form:

```bash
brotli -d <parquet_file>.br
```

For example:

```bash
brotli -d xx_36785.parquet.br
```

On Ubuntu, brotli can be installed through APT:

```bash
sudo apt-get install brotli
```

Schema and Statistics
=

See below for a quick metadata dump for one of the sample parquet files: `xx_36785.parquet`.

```
File path:  xx_36785.parquet
Created by: parquet-cpp-arrow version 13.0.0
Properties: (none)
Schema:
message schema {
  required int32 element_id (INTEGER(32,true));
  required int32 vertex_id (INTEGER(32,true));
  required double v_x;
  required double v_y;
  required double v_z;
  required double rho;
  required double e;
  required double x;
  required double y;
  required double z;
}


Row group 0:  count: 4194304  24.24 B records  start: 4  total(compressed): 96.949 MB total(uncompressed):288.018 MB 
--------------------------------------------------------------------------------
            type      encodings count     avg size   nulls   min / max
element_id  INT32     S   _     4194304   0.88 B     0       "0" / "524287"
vertex_id   INT32     S   _     4194304   2.03 B     0       "0" / "612310"
v_x         DOUBLE    S   _     4194304   2.79 B     0       "-5.0671622E-4" / "1.21735"
v_y         DOUBLE    S   _     4194304   2.78 B     0       "-0.39468541" / "0.39289976"
v_z         DOUBLE    S   _     4194304   2.75 B     0       "-0.3974602" / "0.38954242"
rho         DOUBLE    S   _     4194304   3.98 B     0       "0.12228425" / "4.2933597"
e           DOUBLE    S   _     4194304   3.96 B     0       "0.18369001" / "3.8787233"
x           DOUBLE    S   _     4194304   1.84 B     0       "-0.0" / "7.0"
y           DOUBLE    S   _     4194304   1.68 B     0       "-0.0" / "3.0"
z           DOUBLE    S   _     4194304   1.55 B     0       "-0.0" / "3.0"
```

Sample DuckDB SQL Query
=

```sql
SELECT min(vertex_id) AS VID, min(x) as X, min(y) as Y, min(z) as Z, avg(e) AS E
FROM 'xx_36785.parquet'
WHERE x > 1.5 AND x < 1.6 AND y > 1.5 AND y < 1.6 AND z > 1.5 AND z < 1.6
GROUP BY vertex_id ORDER BY E;
```

## Query Result

```
┌────────┬────────────────────┬────────────────────┬────────────────────┬────────────────────┐
│  VID   │         X          │         Y          │         Z          │         E          │
│ int32  │       double       │       double       │       double       │       double       │
├────────┼────────────────────┼────────────────────┼────────────────────┼────────────────────┤
│ 166462 │          1.5645773 │ 1.5376104999999998 │ 1.5803627999999998 │ 1.4755714125000001 │
│ 240662 │ 1.5611519000000003 │ 1.5749507000000003 │ 1.5417632999999997 │       1.4774999125 │
│ 219780 │ 1.5264729999999997 │          1.5887712 │          1.5934969 │       1.4871300875 │
│ 240660 │           1.503907 │ 1.5975798999999997 │ 1.5078072999999999 │       1.4890849375 │
│ 159440 │          1.5057572 │          1.5352168 │           1.556996 │         1.49520745 │
│ 233498 │          1.5047735 │          1.5526603 │          1.5391836 │         1.49543775 │
│ 212414 │          1.5057572 │          1.5352168 │           1.556996 │        1.495636475 │
│ 212409 │          1.5047735 │          1.5526603 │          1.5391836 │ 1.4960388500000001 │
└────────┴────────────────────┴────────────────────┴────────────────────┴────────────────────┘
```

VisIt Graphics (so we get a sense of how the data looks like)
=

Results of using LLNL's [VisIt](https://visit-dav.github.io/visit-website/index.html) on the dataset's energy (e), density (rho), and velocity (v_x, v_y, v_z) columns.

| Energy (e) | Density (rho) | Velocity (v_x, v_y, v_z) |
| :---: | :---: | :---: |
| ![energy](visit_e.png) | ![density](visit_rho.png) | ![velocity](visit_v.png) |

Acknowledgement
=

Laghos (LAGrangian High-Order Solver) is a miniapp that solves the time-dependent Euler equations of compressible gas dynamics in a moving Lagrangian frame using unstructured high-order finite element spatial discretization and explicit high-order time-stepping. It is built on top of a general discretization library, [MFEM](http://mfem.org).

The Laghos miniapp is part of the [CEED software suite](http://ceed.exascaleproject.org/software), a collection of software benchmarks, miniapps, libraries, and APIs for efficient exascale discretizations based on high-order finite element and spectral element methods.

The CEED research is funded by the U.S. Department of Energy/Exascale Computing Project, an effort responsible for the planning and preparation of a capable exascale ecosystem, including software, applications, hardware, advanced system engineering, and early testbed platforms, in support of the nation’s exascale computing imperative.

VisIt is an open-source, interactive, scalable, visualization, animation, and analysis tool developed by Lawrence Livermore National Laboratory.
VisIt is supported by the U.S. Department of Energy with fundings from the Advanced Simulation and Computing Program, the Scientific Discovery through Advanced Computing Program, and the Exascale Computing Project.

[DuckDB](https://duckdb.org/) is an open source high-performance analytical database system.

This sample dataset is prepared by an employee of Triad National Security, LLC which operates Los Alamos National Laboratory for the U.S. Department of Energy/National Nuclear Security Administration.

Appendix
=

The command we used to run Laghos was `mpirun -np 32 ./laghos -p 3 -m data/box01_hex.mesh -rs 2 -rp 3 -tf 5.0 -pa -visit -print -k <datadir>`.

Detailed per-column page-level information of the sample parquet file: [sample-page-level-info.txt](sample-page-level-info.txt).

Thanks!


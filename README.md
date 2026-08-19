# seqpy_tools

A collection of Python utility functions for working with sequencing files, FASTQ files, sample IDs, file compression, file pairing, subprocesses, logging, and spreadsheet input.

The package is primarily designed to simplify common tasks when processing sequencing data, particularly FASTQ/FASTA files and paired-end sequencing reads.

## Installation

Install the package from PyPI using:

```bash
pip install seqpy_tools
```

### Dependencies

The package requires:

* [NumPy](https://numpy.org/)
* [Pandas](https://pandas.pydata.org/)
* [SciPy](https://scipy.org/)
* [pgzip](https://github.com/madler/pigz)

These dependencies will normally be installed automatically when you install the package with `pip`.

## Importing the functions

The functions can be imported from the package, for example:

```python
from seqpy_tools import find_paired_ids
```

If the functions are not exposed through `__init__.py`, they can instead be imported directly from the functions module:

```python
from seqpy_tools.functions import find_paired_ids
```

---

# Functions

## `check_and_handle_gunzipped()`

Checks a directory for uncompressed FASTQ files and either compresses or removes them.

If an uncompressed file already has a corresponding `.gz` file, the uncompressed file is deleted.

If no compressed version exists, the file is compressed using `pgzip` and the original file is removed.

### Usage

```python
check_and_handle_gunzipped(input_dir)
```

### Parameters

| Parameter   | Type  | Description                          |
| ----------- | ----- | ------------------------------------ |
| `input_dir` | `str` | Directory containing the FASTQ files |

### Example

```python
from seqpy_tools import check_and_handle_gunzipped

check_and_handle_gunzipped("/data/reads")
```

For example, if the directory contains:

```text
sample1.fastq
sample1.fastq.gz
sample2.fastq
```

`sample1.fastq` will be removed because its compressed version already exists.

`sample2.fastq` will be compressed to:

```text
sample2.fastq.gz
```

and the original `sample2.fastq` will then be removed.

---

## `clean_and_tar()`

Removes a `tmp` directory from an output directory and creates a gzipped tar archive of an input directory.

The resulting archive is placed in the parent directory of the input directory.

### Usage

```python
clean_and_tar(output_directory, input_directory)
```

### Parameters

| Parameter          | Type  | Description                                    |
| ------------------ | ----- | ---------------------------------------------- |
| `output_directory` | `str` | Directory in which a `tmp` directory may exist |
| `input_directory`  | `str` | Directory to archive                           |

### Example

```python
from seqpy_tools import clean_and_tar

clean_and_tar(
    output_directory="/data/output",
    input_directory="/data/output/raw_reads"
)
```

This creates:

```text
/data/output/raw_reads.tar.gz
```

and removes:

```text
/data/output/tmp/
```

if it exists.

---

## `compress_file()`

Compresses a single file using gzip.

Files that already end in `.gz` are ignored.

The original uncompressed file is removed after successful compression.

### Usage

```python
compress_file(filepath)
```

### Parameters

| Parameter  | Type  | Description                  |
| ---------- | ----- | ---------------------------- |
| `filepath` | `str` | Path to the file to compress |

### Example

```python
from seqpy_tools import compress_file

compress_file("/data/reads/sample.fastq")
```

This produces:

```text
sample.fastq.gz
```

and removes:

```text
sample.fastq
```

---

## `find_program()`

Checks whether an external command-line program is available in the system `PATH`.

This is useful when your workflow requires external software such as `repair.sh`.

### Usage

```python
find_program(program_name)
```

### Parameters

| Parameter      | Type  | Description                          |
| -------------- | ----- | ------------------------------------ |
| `program_name` | `str` | Name of the executable to search for |

### Example

```python
from seqpy_tools import find_program

repair_path = find_program("repair.sh")
```

If the program is found, its path is returned.

If it cannot be found, an error is logged and the program exits.

---

## `find_single_reads()`

Finds single-end FASTQ files in a directory.

Files identified as paired-end reads, such as `_1`, `_2`, `_R1`, or `_R2`, are excluded.

### Usage

```python
find_single_reads(
    input_dir,
    prefix=None,
    suffix=None
)
```

### Parameters

| Parameter   | Type            | Description                                                |
| ----------- | --------------- | ---------------------------------------------------------- |
| `input_dir` | `str`           | Directory to search                                        |
| `prefix`    | `str` or `list` | Optional filename prefix filter                            |
| `suffix`    | `str`           | Optional suffix that must occur before the FASTQ extension |

### Example

```python
from seqpy_tools import find_single_reads

files = find_single_reads("/data/reads")

print(files)
```

You can filter by prefix:

```python
files = find_single_reads(
    "/data/reads",
    prefix="sample"
)
```

Multiple prefixes can be supplied as a comma-separated string:

```python
files = find_single_reads(
    "/data/reads",
    prefix="sample1,sample2"
)
```

You can also specify a suffix:

```python
files = find_single_reads(
    "/data/reads",
    suffix="merged"
)
```

The function returns a list of matching file paths.

---

## `get_ids()`

Extracts sample IDs from either a spreadsheet or a directory containing sequencing files.

For spreadsheet input, IDs are extracted from a specified column.

For directory input, IDs are inferred from FASTQ filenames.

### Usage

```python
get_ids(
    input_source,
    column_name=None,
    sheet=None
)
```

### Parameters

| Parameter      | Type           | Description                                     |
| -------------- | -------------- | ----------------------------------------------- |
| `input_source` | `str`          | Path to a CSV/XLSX file or sequencing directory |
| `column_name`  | `str`          | Column containing IDs when using a spreadsheet  |
| `sheet`        | `str` or `int` | Optional Excel sheet                            |

### Example: spreadsheet

```python
from seqpy_tools import get_ids

ids = get_ids(
    "/data/samples.csv",
    column_name="sample_id"
)

print(ids)
```

For an Excel file:

```python
ids = get_ids(
    "/data/samples.xlsx",
    column_name="sample_id",
    sheet="Sheet1"
)
```

### Example: directory

```python
ids = get_ids("/data/reads")

print(ids)
```

The function returns a list of sample IDs.

---

## `get_read_ids()`

Returns sample IDs from a directory or spreadsheet.

It can identify:

* all samples
* paired-end samples
* single-end samples

### Usage

```python
get_read_ids(
    input_source,
    mode=None,
    prefix=None,
    suffix=None,
    column_name=None,
    sheet=None
)
```

### Parameters

| Parameter      | Type   | Description                              |
| -------------- | ------ | ---------------------------------------- |
| `input_source` | `str`  | Directory or CSV/XLSX file               |
| `mode`         | `str`  | `"paired"`, `"single"`, or `None`        |
| `prefix`       | `list` | Optional filename prefixes               |
| `suffix`       | `str`  | Optional filename suffix                 |
| `column_name`  | `str`  | Spreadsheet column containing sample IDs |
| `sheet`        | `str`  | Optional Excel sheet                     |

### Find all IDs

```python
ids = get_read_ids("/data/reads")
```

### Find paired samples

```python
paired_ids = get_read_ids(
    "/data/reads",
    mode="paired"
)
```

### Find single-end samples

```python
single_ids = get_read_ids(
    "/data/reads",
    mode="single"
)
```

### Spreadsheet input

```python
ids = get_read_ids(
    "/data/samples.xlsx",
    column_name="sample_id",
    sheet="Sheet1"
)
```

The function returns a sorted list of sample IDs.

---

## `gzip_files_in_dir()`

Compresses all uncompressed files in a directory.

Compression is performed in parallel using multiple worker threads.

### Usage

```python
gzip_files_in_dir(
    input_dir,
    max_workers=4
)
```

### Parameters

| Parameter     | Type  | Description                                    |
| ------------- | ----- | ---------------------------------------------- |
| `input_dir`   | `str` | Directory containing files to compress         |
| `max_workers` | `int` | Maximum number of parallel compression workers |

### Example

```python
from YOUR_PACKAGE_NAME import gzip_files_in_dir

gzip_files_in_dir("/data/reads")
```

To increase the number of parallel workers:

```python
gzip_files_in_dir(
    "/data/reads",
    max_workers=8
)
```

Files that already end in `.gz` are skipped.

---

## `pair_input_files()`

Finds paired sequencing files and groups them into pairs.

The function recognises common paired-end naming conventions including:

```text
sample_1.fastq
sample_2.fastq
```

and:

```text
sample_R1.fastq.gz
sample_R2.fastq.gz
```

### Usage

```python
pair_input_files(
    input_directory,
    prefixes=None,
    suffix=None
)
```

### Parameters

| Parameter         | Type            | Description                                |
| ----------------- | --------------- | ------------------------------------------ |
| `input_directory` | `str`           | Directory containing sequencing files      |
| `prefixes`        | `str` or `list` | Optional filename prefixes                 |
| `suffix`          | `str`           | Optional suffix occurring before `_1`/`_2` |

### Example

```python
from seqpy_tools import pair_input_files

pairs = pair_input_files("/data/reads")

print(pairs)
```

The function returns a list containing pairs of file paths:

```python
[
    [
        "/data/reads/sample1_R1.fastq.gz",
        "/data/reads/sample1_R2.fastq.gz"
    ],
    [
        "/data/reads/sample2_R1.fastq.gz",
        "/data/reads/sample2_R2.fastq.gz"
    ]
]
```

### Filtering by prefix

```python
pairs = pair_input_files(
    "/data/reads",
    prefixes=["sample1", "sample2"]
)
```

### Filtering by suffix

```python
pairs = pair_input_files(
    "/data/reads",
    suffix="_unmerged"
)
```

---

## `pair_input_files_deprecated()`

An older version of `pair_input_files()`.

This function is retained for backwards compatibility but should generally not be used for new code.
For new projects, use `pair_input_files()` instead.

---

## `find_paired_ids()`

Finds paired FASTQ files and returns them grouped by sample ID.

The function searches for `.fastq`, `.fq`, `.fastq.gz`, and `.fq.gz` files.

### Usage

```python
find_paired_ids(
    input_dir,
    prefixes=None,
    suffix=None
)
```

### Parameters

| Parameter   | Type   | Description                                |
| ----------- | ------ | ------------------------------------------ |
| `input_dir` | `str`  | Directory containing sequencing files      |
| `prefixes`  | `list` | Optional filename prefixes                 |
| `suffix`    | `str`  | Optional suffix appearing before `_1`/`_2` |

### Example

```python
from seqpy_tools import find_paired_ids

paired_reads = find_paired_ids("/data/reads")
```

The function returns a dictionary:

```python
{
    "sample1": (
        "/data/reads/sample1_R1.fastq.gz",
        "/data/reads/sample1_R2.fastq.gz"
    ),
    "sample2": (
        "/data/reads/sample2_R1.fastq.gz",
        "/data/reads/sample2_R2.fastq.gz"
    )
}
```

This can be useful when you need both the sample ID and the corresponding read files.

---

## `repair_reads()`

Repairs paired-end sequencing reads using the external `repair.sh` program from BBMap.

The function creates repaired paired reads and removes the resulting single reads.

### Important

This function requires `repair.sh` to be installed separately and available in your system `PATH`.

The Python package does **not** install BBMap.

### Usage

```python
repair_reads(
    read1,
    read2,
    input_directory,
    sample_id,
    logger
)
```

### Parameters

| Parameter         | Type             | Description                        |
| ----------------- | ---------------- | ---------------------------------- |
| `read1`           | `str`            | Path to read 1                     |
| `read2`           | `str`            | Path to read 2                     |
| `input_directory` | `str`            | Output directory                   |
| `sample_id`       | `str`            | Sample identifier                  |
| `logger`          | `logging.Logger` | Logger used for reporting progress |

### Example

```python
import logging

from seqpy_tools import repair_reads

logger = logging.getLogger(__name__)

repair_reads(
    read1="/data/reads/sample1_R1.fastq.gz",
    read2="/data/reads/sample1_R2.fastq.gz",
    input_directory="/data/reads",
    sample_id="sample1",
    logger=logger
)
```

The function calls:

```text
repair.sh
```

and generates repaired paired files in the input directory.

---

## `run_command()`

Runs an external command and saves its standard output and standard error to separate log files.

### Usage

```python
run_command(
    command,
    log_prefix,
    log_dir="./logs"
)
```

### Parameters

| Parameter    | Type   | Description                       |
| ------------ | ------ | --------------------------------- |
| `command`    | `list` | Command and arguments to execute  |
| `log_prefix` | `str`  | Prefix used for the log filenames |
| `log_dir`    | `str`  | Directory in which to save logs   |

### Example

```python
from seqpy_tools import run_command

run_command(
    ["echo", "Hello world"],
    log_prefix="example"
)
```

This creates:

```text
logs/
├── example_output.log
└── example_error.log
```

The command itself is also written to the log files.

If the command fails, `subprocess.run()` raises an exception.

---

## `run_subprocess()`

Runs an external command and captures its output.

Standard output and standard error are written to separate log files.

### Usage

```python
run_subprocess(
    command,
    log_prefix,
    log_dir="./logs"
)
```

### Parameters

| Parameter    | Type   | Description                        |
| ------------ | ------ | ---------------------------------- |
| `command`    | `list` | Command and arguments to execute   |
| `log_prefix` | `str`  | Prefix used for the log filenames  |
| `log_dir`    | `str`  | Directory containing the log files |

### Example

```python
from seqpy_tools import run_subprocess

run_subprocess(
    ["echo", "Hello world"],
    log_prefix="example",
    log_dir="./logs"
)
```

The output will be written to:

```text
logs/example_output.log
```

and:

```text
logs/example_error.log
```

If the command returns a non-zero exit code, a `CalledProcessError` is raised.

---

## `setup_logging()`

Sets up logging to both a file and the console.

The file receives DEBUG-level messages and above.

The console receives INFO-level messages and above.

### Usage

```python
setup_logging(
    log_dir="./logs",
    log_file="log.out"
)
```

### Parameters

| Parameter  | Type  | Description                                    |
| ---------- | ----- | ---------------------------------------------- |
| `log_dir`  | `str` | Directory where the log file should be created |
| `log_file` | `str` | Name of the log file                           |

### Example

```python
from seqpy_tools import setup_logging

logger = setup_logging(
    log_dir="./logs",
    log_file="analysis.log"
)

logger.info("Analysis started")
logger.debug("Debug information")
logger.warning("Example warning")
```

This creates:

```text
logs/
└── analysis.log
```

---

## `xlsx2csv()`

Converts an Excel `.xlsx` file into a CSV file using Pandas.

An optional Excel sheet can be selected.

### Usage

```python
xlsx2csv(
    file_path,
    sheet=None
)
```

### Parameters

| Parameter   | Type           | Description                  |
| ----------- | -------------- | ---------------------------- |
| `file_path` | `str`          | Path to the XLSX file        |
| `sheet`     | `str` or `int` | Optional sheet name or index |

### Example

Convert the first/default sheet:

```python
from seqpy_tools import xlsx2csv

csv_file = xlsx2csv(
    "/data/samples.xlsx"
)

print(csv_file)
```

Specify a sheet:

```python
csv_file = xlsx2csv(
    "/data/samples.xlsx",
    sheet="Samples"
)
```

The resulting CSV is written in the same directory as the original Excel file.

The function returns the path to the generated CSV file.

---

# Typical workflow

The functions can be combined to create a sequencing data-processing workflow.

For example, you could first find paired reads:

```python
from seqpy_tools import find_paired_ids

paired_reads = find_paired_ids("/data/reads")
```

Then process each sample:

```python
for sample_id, (read1, read2) in paired_reads.items():
    print(sample_id)
    print(read1)
    print(read2)
```

You could then repair the reads:

```python
from seqpy_tools import repair_reads

for sample_id, (read1, read2) in paired_reads.items():
    repair_reads(
        read1,
        read2,
        "/data/reads",
        sample_id,
        logger
    )
```

---

# External software

Some functions rely on software outside of Python. In particular:

### BBMap

`repair_reads()` requires the BBMap `repair.sh` executable.

BBMap must be installed separately and `repair.sh` must be available in the system `PATH`. You can check whether it is available with:

```python
from seqpy_tools import find_program

find_program("repair.sh")
```

If the program is not available, the function will report an error.

---

# Supported Python versions

This package is intended to support Python 3.12 and later.

---

# License

This project is licensed under the MIT License. See the [LICENSE]([LICENSE](https://github.com/museomics/seqpy_tools/edit/main/LICENSE)) file for details.

# Python

## Custom Mini Library

Below is a collection of small subroutines I use frequently in my system scripts. 

## Argparse

```python
import argparse

def parse_args():
    parser = argparse.ArgumentParser()

    parser.add_argument("-d", "--debug", action="store_true", help="Enable debug output.")
    parser.add_argument("somestring", help="A string argument.")
    parser.add_argument("someint", type=int, help="An integer argument.")

    return parser.parse_args()

```

## Subprocess

```python

import subprocess


def run_command(command):
    result = subprocess.run(
        command,
        capture_output=True,
        text=True,
        check=True,
    )
    return result.stdout
```
---
title: "Packaging a Simple Python Script with PEX"
tags: python pex
---

Recently I needed to create a package for a simple Python CLI tool which includes all dependencies.
It turns out that there is a great tool [PEX](https://pex.readthedocs.org/en/stable/) which can build
an executable zip Python package.

That's exactly what I needed, but unfortunately the documentation for my simple use case was sorely lacking. Described here is how to get PEX working to package a simple CLI tool.

ENDOFSUMMARY

## Requirements

Using Python3 and PEX as installed by `pip`

## Structure

Key factors
* setup.py is required to allow building a wheel for the CLI tool
* source must be in a package directory (even if it's only one file)

<pre>
/setup.py
/samplepkg/
/samplepkg/__init__.py
/samplepkg/main.py
</pre>

### setup.py

The `setup.py` contains a package description. More details can be found [here](https://docs.python.org/3.5/distutils/setupscript.html).

The following is a minimal file.

<%= render_code("setup.py", "python") %>

### __init__.py

Empty file to mark `samplepkg` as a package directory

### main.py

The actual CLI code entry point

<%= render_code("samplepkg/main.py", "python") %>

## Build

PEX requires first that we build a wheel from the sources, then include that wheel in the PEX build

### Wheel Build

In the directory containing setup.py run

``` bash
pip3 wheel -w . .
```

This will create a wheel in the current directory based on the `name` set in `setup.py`.
In this example the file generated is

`myexample-0.0.0-py3-none-any.whl`

### PEX Build

Now we build the PEX package.

``` bash
pex --python=python3 -f $PWD requests myexample -e samplepkg.main -o samplepkg.pex
```

Breakdown:

* `-f $PWD` - Causes the current directory to be included in the search path, which allows the `myexample-*.whl` to be loaded
* `requests` - Added dependency on the `requests` package. Any other dependencies can be added as extra arguments here
* `myexample` - Loads the wheel that we just built
* `-e samplepkg.main` - The execution command to run, in this case load our `main.py` which starts the CLI
* `-o samplepkg.pex` - Write the whole shindig to the shippable file `samplepkg.pex`

And try running the generated package

``` text
$ ./samplepkg.pex
HELLO WORLD!!
```

## Notes

### Python Version

Both `pex` and `pip wheel` *must* use the same version of Python. Without being explicit about version it's possible for the two to get crossed. If this happens you will get an error running `pex`: "Could not satisfy all requirements" which means that the wheel file couldn't be found for the current Python version. You can tell what version was used for `pip wheel` by the `-pyX-` portion of the wheel file name.

In the code above I explicitly selected Python 3 to be used.
To use Python 2 instead, edit the commands above to replace `pip3` with `pip2` and `pex --python=python3` with `pex --python-python2`


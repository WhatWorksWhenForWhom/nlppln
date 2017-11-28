# NLP Pipeline

A python package to create NLP pipelines using [Common Workflow Language](http://www.commonwl.org/) (CWL).

Basically, it provides python scripts for common NLP tasks that can be run as
command line tools, and CWL specifications to use those tools. Most tools
wrap existing NLP functionality.
The command line tools are made with [Click](http://click.pocoo.org), a Python
package for creating command line interfaces.

**Please note**: CWL is not yet compatible with
[Python 3](https://github.com/common-workflow-language/cwltool/issues/310) and
[Windows](https://github.com/common-workflow-language/cwltool/issues/340).

## Installation

Install [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git),
[Python](https://www.python.org/downloads/) 2.7 and [pip](https://pip.pypa.io/en/stable/installing/). (You also may need to install
  setuptools (`pip install setuptools`)).

We recommend installing `nlppln` in a
[virtual environment](https://virtualenv.pypa.io/en/stable/) (`pip install virtualenv`).

```
pip install nlppln
```

Tools can be run by using the Python -m option, e.g. `python -m nlppln.commands.apachetika <INPUTDIR> <OUTPUTDIR>`.

The CWL specifications of tools and workflows are not included if you install
`nlppln` with pip. If you want to run tools and workflows, please download the
(top-level) [cwl directory](https://github.com/WhatWorksWhenForWhom/nlppln/tree/master/cwl)
and put them somewhere on your machine.

To run CWL workflows created with `nlppln`, install a cwl-runner (`pip install
cwlref-runner`) and [Docker](https://docs.docker.com/engine/installation/).

### For development

```
git clone https://github.com/WhatWorksWhenForWhom/nlppln.git
cd nlppln

virtualenv /path/to/env
source /path/to/env/bin/activate

git checkout develop
pip install -r requirements.txt
python setup.py develop
```

When installing in development mode, you don't need to downaload the CWL
specifications of tools and workflows separately, because they are included in the
source code. They can be found in the top-level [cwl directory](https://github.com/WhatWorksWhenForWhom/nlppln/tree/master/cwl) .

## Generating command line NLP tool boilerplate and cwl steps

NLP Pipeline contains functionality to generate command line NLP tools and CWL
steps. To generate a command line tool and/or CWL step run:

    python -m nlppln.generate

This command starts a command line interface that asks you to specify the inputs and outputs of the tool:

```
> python -m nlppln.generate
Generate python command? [y]:
Generate cwl step? [y]:
Command name [command]:
Metadata input file? [n]: y
Multiple input files? [y]:
Multiple output files? [y]:
Extension of output files? [json]:
Metadata output file? [n]: y
Save python command to [nlppln/command.py]:
Save metadata to? [metadata_out.csv]:
Save cwl step to [cwl/steps/command.cwl]:
```

## Creating workflows

Workflows can be created by writing a Python script.

```python
from nlppln import WorkflowGenerator

wf = WorkflowGenerator()
wf.load(steps_dir='/path/to/dir/with/cwl/steps/')

txt_dir = wf.add_inputs(txt_dir='Directory')

frogout = wf.frog_dir(dir_in=txt_dir)
saf = wf.frog_to_saf(in_files=frogout)
ner_stats = wf.save_ner_data(in_files=saf)
new_saf = wf.replace_ner(metadata=ner_stats, in_files=saf)
txt = wf.saf_to_txt(in_files=new_saf)

wf.add_outputs(ner_stats=ner_stats, txt=txt)

wf.save('anonymize.cwl')
```

Additional processing steps can be loaded using:

```python
from nlppln import WorkflowGenerator

wf = WorkflowGenerator()
wf.load(steps_dir='/path/to/dir/with/cwl/steps/')
```

To load a single cwl file, do:
```python
wf.load(step_file='/path/to/step_or_workflow.cwl')
```

See [scriptcwl](https://github.com/NLeSC/scriptcwl) for more information on creating
workflows.

## Workflows

### Anonymize

The anonmize-workflow finds named entities in all text files in a directory. Named entities
are replaced with their type (PER, LOC, ORG). The output consists of text files and a csv file that contains the named entities that have been replaced.

Usage:
```
cwl-runner /path/to/nlppln/cwl/anonymize.cwl --txt-dir /path/to/dir/with/text/files
```
The text files with named entities removed are saved in the directory where the pipeline is run.

## GUI

NLP Pipeline provides a GUI to inspect the results of running text processing workflows.
Currently, the GUI allows users to inspect the results of named entity recognition.

Command:

    python -m nlppln.commands.inspect_ne <META IN> <IN FILES>

Results can be inspected at http://localhost:5000/ (the browser is started automatically).
For development, start the GUI with `python -m nlppln.gui.server <META IN> <IN FILES>`.
In this case, the GUI is started with `debug=True`.

Please note that the GUI requires some additional work (see [Installation](#installation)).

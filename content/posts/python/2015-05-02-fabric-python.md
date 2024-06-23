+++
title = "Installing Python 3 compatible Fabric"
date = 2015-05-02T23:12:13+02:00
categories = ['python']
author = "Morten Kjeldgaard"
+++

Unfortunately, [Fabric](https://pypi.python.org/pypi/Fabric/1.10.1)
has not yet officilly been ported to Python 3. However, it is possible
to get it to work. Here is how I did it.

First, create a virtual environment to run fabric in. (If you haven't
set up [virtual environments](https://virtualenv.pypa.io/en/latest/)
for Python, go ahead and do that now, then come back.) I have the
[fink environment] (http://finkproject.org) installed on my Mac, and
the Python 3 executable is in `/sw/bin/python3`. If you have Python3
from homebrew or elsewhere, just put the correct path to the python3
executable after the `--python` switch below:

    $ mkvirtualenv --python=/sw/bin/python3 fabric
    [ ... ]

Save the following lines in a file `/tmp/requirements.txt`:

    Babel==1.3
    Jinja2==2.7.3
    MarkupSafe==0.23
    Pygments==2.0.2
    Sphinx==1.3.1
    alabaster==0.6.1
    docutils==0.12
    ecdsa==0.13
    fudge==0.9.6
    invocations==0.10.0
    invoke==0.10.1
    nose==1.3.6
    paramiko==1.15.2
    pycrypto==2.6.1
    pytz==2015.2
    releases==0.6.1
    semantic-version==2.4.0
    six==1.9.0
    snowballstemmer==1.2.0
    sphinx-rtd-theme==0.1.7
    wheel==0.24.0

- - -
[Added 20150509] If you don't care about running fabric's unittests, you can actually get by with this list:

    ecdsa
    paramiko
    six
- - -

In the `fabric` environment, install these packages:

    (fabric)$ pip3 install -r /tmp/requirements.txt

Now, all you need is the last two requirements, which are not
available from [PyPi](https://pypi.python.org/), because they are
forked from the main developement line. Go to the place you store
your github clones:

    (fabric)$ cd ~/github

First, clone and install the Python 3 version of `rudolf`:

    (fabric)$ git clone --depth=1 git@github.com:mok0/rudolf.git
    (fabric)$ cd rudolf
    (fabric)$ python3 setup.py install

(The `--depth=1` switch gives a shallow clone. It is sufficient for most
cases, unless you are interested in the history of the repo.)
Next, clone the git repository from [Sergey Pashinin](https://github.com/pashinin), who has
ported fabric itself to Python 3:

    (fabric)$ cd ~/github
    (fabric)$ git clone --depth=1 git@github.com:pashinin/fabric.git
    (fabric)$ cd fabric

See the tests, most of them run ok, but a few do fail :-(

    (fabric)$ python3 setup.py nosetests

Even though some tests fail, fabric seems to work fine for most
things I have tried. Build and install:

    (fabric)$ python3 setup.py install

Now, test out your `fabric` installation. From the documentation,
try the following example. Place in `/tmp/fabfile.py`:

    from fabric.api import run
    def host_type():
        run('uname -s')

Now try to run it on `localhost` (requires ssh):

    (fabric)$ fab -H localhost host_type
    [localhost] Executing task 'host_type'
    [localhost] run: uname -s
    [localhost] out: Darwin
    [localhost] out:

    Done.
    Disconnecting from localhost... done.
    (fabric)$

Deactivate the `fabric` enviroment:

    (fabric)$ deactivate
    $

Whenever you need to use your `fabric` installation, just issue the command:

    $ workon fabric

(The latter requires that you have also installed virtualenvwrapper).

That's all, good luck!

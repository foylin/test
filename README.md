# Getting Started

To get started, read [STARTED](STARTED.md) document.

Below is the template of the board's `README.md`.

### Firefly-RK3399 Manual

This is the manual of the Firefly-RK3399 board.

The manual can be built by:

```bash
# Install requiremnts
sudo apt-get install virtualenv
virtualenv --python=python3 ~/sphinx-markdown
source ~/sphinx-markdown/bin/activate
pip install -r requirements.txt

# Build English Manual
make -C en html
# open en/_build/html/index.html in browser

# Build Chinese Manual
make -C zh_CN html
# open zh_CN/_build/html/index.html in browser
```

`source ~/sphinx-markdown/bin/activate` needs to run once before the `make` commands, or you can run them in a subshell:

> (source ~/sphinx-markdown/bin/activate && make -C en html)

---
(Tested in Ubuntu 16.04)

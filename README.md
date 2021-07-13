prepare environment:

```bash
# create python3 virtual environment
python3 -m venv env

# source it
source env/bin/activate

# install dependencies
pip install -r requirements.txt

# deacitave environment
deactivate

```

To compile to html:
```bash
source env/bin/activate
make clean html

# output should be under build/html

```
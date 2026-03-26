It's common to have multiple Python applications running on your system.

When applications requires the same module, at some point you will reach a situation where an app needs a version of a module, and another app requires a different version of that same module.

To solve this, you can use **virtual environments**.

A **virtual environment** solves this by giving each project its own:
- Python interpreter (linked, not fully copied)
- Installed libraries
- Dependencies

#### What a virtual environment contains
When you create one, Python makes a folder with:
- `python` executable
- `pip`
- `site-packages` (where libraries go)

#### How to create and use one
##### 1. Create a virtual environment
```bash
python -m venv myenv
```

##### 2. Activate it
- windows
```bash
myenv\Script\activate
```
- Max/Linux
```bash
source myenv/bin/activate
```
After activation, your terminal will look like:
```bash
(myenv) $
```
##### 3. Install packages (isolated)
```bash
pip install requests
```
This installs **only inside the virtual environment**, not globally.

##### 5. Deactivate
```bash
deactivate
```

# Configuring post-save hooks

Sometimes it helps to commit the .ipynb, .py, and .html of every notebook in each commit. Creating the .py and .html files can be done simply and automatically every time a notebook is saved by editing the jupyter config file and adding a post-save hook.

The default jupyter config file is found at:

    ~/.jupyter/jupyter_notebook_config.py

If you don’t have this file, run: 

    jupyter notebook --generate-config 

to create this file, and add the following text:

```python
import io
import os
from notebook.utils import to_api_path
_script_exporter = None
_html_exporter = None
def script_post_save(model, os_path, contents_manager, **kwargs):
    """convert notebooks to Python script after save with nbconvert

    replaces `ipython notebook --script`
    """
    from nbconvert.exporters.script import ScriptExporter

    if model['type'] != 'notebook':
        return

    global _script_exporter

    if _script_exporter is None:
        _script_exporter = ScriptExporter(parent=contents_manager)

    log = contents_manager.log

    base, ext = os.path.splitext(os_path)
    py_fname = base + '.py'
    script, resources = _script_exporter.from_filename(os_path)
    script_fname = base + resources.get('output_extension', '.txt')
    log.info("Saving script /%s", to_api_path(script_fname, contents_manager.root_dir))

    with io.open(script_fname, 'w', encoding='utf-8') as f:
        f.write(script)

    """
    Also convert notebooks to HTML after save with nbconvert
    """
    from nbconvert.exporters.html import HTMLExporter

    if model['type'] != 'notebook':
        return

    global _html_exporter

    if _html_exporter is None:
        _html_exporter = HTMLExporter(parent=contents_manager)

    log = contents_manager.log

    base, ext = os.path.splitext(os_path)

    html, resources = _html_exporter.from_filename(os_path)
    html_fname = base + resources.get('output_extension', '.html')
    log.info("Saving HTML /%s", to_api_path(html_fname, contents_manager.root_dir))

    with io.open(html_fname, 'w', encoding='utf-8') as f:
        f.write(html)

c.FileContentsManager.post_save_hook = script_post_save
```
> Adapted from http://jupyter-notebook.readthedocs.io/en/latest/extending/savehooks.html#examples
and http://www.kdnuggets.com/2016/10/jupyter-notebook-best-practices-data-science.html
with reference to https://github.com/jupyter/nbconvert and http://nbconvert.readthedocs.io/en/latest/api/exporters.html

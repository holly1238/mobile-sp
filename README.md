# PyTorch Mobile 
This site is built in Sphinx. You can build it locally by cloning the repo
and running:

```
pip install -r requirements.txt
make html
```

The document set will be built in the build/html subfolder. You can preview
it by opening the index.html file on your local machine, or by running a
server such as `python -m http.server`

The toc for this site is contained in the source/contents.rst file. 

This repo contains a github workflow script that builds out the documentation and
copies the output to gh-pages branch. This branch is used to render the
site through github pages. 

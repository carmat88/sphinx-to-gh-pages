# Sphinx to GitHub Pages
This is a quick, simple repo for testing GitHub actions. Specifically, to test how to publish documentation created via Sphinx onto GitHub pages (that is pages within the github.io domain).

- Sphinx Getting started documentation: https://www.sphinx-doc.org/en/master/tutorial/getting-started.html
- Here is the GitHub actions documentation: https://docs.github.com/en/actions
- Pre-built _Sphinx to GitHub Pages_ action being used: https://github.com/marketplace/actions/sphinx-to-github-pages
- A well produced video tutorial: https://www.youtube.com/watch?v=R8_veQiYBjI

# Getting Started
In addition to the GitHub documentation as well as this [pre-built Sphinx action](https://github.com/marketplace/actions/sphinx-to-github-pages) are well written and easy to follow, here comes some steps-by-steps action and an extra tips for achieving this task.

**1. If it does not exist yet, create a repository in GitHub where your Sphinx documentation will exist**

**2. Clone locally such repository, locate yourself within its path (i.e. `cd new-repo`) and start a python virtual environment**
```bash
git clone "my_new_repo"

cd "my_new_repo"

python -m venv my_venv

source my_venv/bin/activate

(my_venv) $ ls
```

**3. First install the pip module for Sphinx**
```bash
(my_venv) $ pip install sphinx
```
   - 3.1 Make sure that Sphinx is correctly installed by running the command:
```bash
(my_venv) $ sphinx-build --version

sphinx-build 4.2.0
```
  - 3.2 Now it is time to generate a Sphinx project as follow (please check further info at the [official Sphinx doc page](https://www.sphinx-doc.org/en/master/tutorial/getting-started.html)):
```
(my_venv) $ sphinx-quickstart docs

(my_ven) $ tree docs

docs
├── build
├── make.bat
├── Makefile
└── source
   ├── conf.py
   ├── index.rst
   ├── _static
   └── _templates
```

**4. Setting up configuration details in `conf.py`**

The configuration directory must contain a file named `conf.py`. This file (containing Python code) is called the “build configuration file” and contains (almost) all configuration needed to customize Sphinx input and output behavior. In addition to these configuration values, you can customize Sphinx even more by using extensions. One of the available extensions that in particular we need is `sphinx.ext.githubpages`. So to enable such extension, locate the `extensions` list in your `conf.py` and add one element as follows:

_docs/source/conf.py_   (open such file with your favorite editor)
```
# Add any Sphinx extension module names here, as strings. They can be
# extensions coming with Sphinx (named 'sphinx.ext.*') or your custom
# ones.
extensions = [
    'sphinx.ext.githubpages',
]
```
Keep modify this file with all changes that you may see fit for your Sphinx documentation. For more details, please refer to its [official documentation](https://www.sphinx-doc.org/en/master/contents.html). In case you would to customize your docs with third-parties themes which extends the Sphinx basic ones, here is a collection of them: https://sphinx-themes.org/

**5. Let's create one last file, commit and push the changes onto your repository**

Before commiting and pushing such changes, we need to create an empty file called `.nojekyll` inside the `docs` folder:
```bash
(my_ven) $ cd docs
(my_ven) $ touch .nojekyll
```
This file simply indicates that we are not using Jekyll as our static site generator in this repository. When GitHub sees a `.nojekyll` file, it serves the root `index.html`. Alright, now it is time to commit and push our changes:
```bash
(my_ven) $ git add .
(my_ven) $ git commit -m "Adding Sphinx docs folder"
(my_ven) $ git push -u origin main
```
At this point in your GitHub UI, if and only if a new fresh repository was created, it should look something like this:

![image](https://user-images.githubusercontent.com/26251075/137332603-271cc737-f9ff-4b9b-a942-1cfdeea3c74c.png)

**6. Creating a GitHub action**

On the GitHub top navigation bar, click on _Actions_ button. If it is the first time setting up an action, you will then be presented with a selection of possible already existing, very common and useful action templates:

![image](https://user-images.githubusercontent.com/26251075/137333921-de411bad-ee4c-42c4-8fd8-c396c284836f.png)


As explained at the beginning of this readme, for this test I have used the following already pre-built action: https://github.com/marketplace/actions/sphinx-to-github-pages. For more details about GitHub actions, please refer to the [GitHub official documentation](https://docs.github.com/en/actions).

Click on the "set up a workflow yourself" link and you will be presented with a basic "Hello World"-style template called `main.yaml`:

![image](https://user-images.githubusercontent.com/26251075/137334403-056c3d13-b68d-4f09-a9da-a5fa9745f0cc.png)

**7. Copy/paste pre-built Sphinx to GitHub Pages template**

As explained at the link of the [pre-built action](https://github.com/marketplace/actions/sphinx-to-github-pages), here comes its original full configured action yaml template:

```yaml
name: Pages
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@master
      with:
        fetch-depth: 0 # otherwise, you will failed to push refs to dest repo
    - name: Build and Commit
      uses: sphinx-notes/pages@master
    - name: Push changes
      uses: ad-m/github-push-action@master
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        branch: gh-pages
```

One could simply replace such template in the content of the autogenerated `main.yaml` above. However I would recommend the following few changes:
- Rename the file from `main.yaml` to something more descriptive and clear, e.g. ``build-sphinx-docs.yaml
- Change the `name:` and `on:` fields within the yaml file to be also more precise and to be triggered on both a push and pull-request:
```yaml
name: Sphinx to GitHub Pages

on:
  # Trigger the workflow on push or pull request,
  # but only for the main branch
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

...

```
- Given the current structure of our Sphinx project (i.e docs/source), we need to tell explicitily to such pre-built action which is the `documentation_path`
```yaml
...

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@master
      with:
        fetch-depth: 0 # otherwise, you will failed to push refs to dest repo
    - name: Build and Commit
      uses: sphinx-notes/pages@master
      with:
        documentation_path: docs/source
      ...
```
So the final yaml file could look something as follows:

```yaml

name: Sphinx to GitHub Pages

on:
  # Trigger the workflow on push or pull request,
  # but only for the main branch
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@master
      with:
        fetch-depth: 0 # otherwise, you will failed to push refs to dest repo
    - name: Build and Commit
      uses: sphinx-notes/pages@master
      with:
        documentation_path: docs/source
    - name: Push changes
      uses: ad-m/github-push-action@master
      with:
        github_token: ${{ secrets.SPHINX_GH_TOKEN }}
        branch: gh-pages
```
**Note:** The generated Sphinx documentation will be pushed into a new branch called _gh-pages_. I will come back to why this is a nice thing to follow.

- **Last but not least:** we need to create a secret in our GitHub repository given line `github_token: ${{ secrets.SPHINX_GH_TOKEN }}`. Please refer to this tutorial about [GitHub Encryted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) before proceding to the next step. The name of the secret does not have to be `SPHINX_GH_TOKEN`. That was just my choice because I find it clear for me. The most important thing is that whichever name you will choose it must match the content of the used yaml variable (i.e `github_token: ${{ secrets.PICK_YOUR_NAME }}`)

**8. Commit the action template**

Once you have correctly configured the Sphinx pre-built template, created the GitHub secret for such action, it is time then to commit such new action by presssing the green _start commit_ button on the right side of the Action UI:

![image](https://user-images.githubusercontent.com/26251075/137338897-b9c6d0fa-4d00-4a20-a1e8-06a29855aa83.png)

Once you have committed directly via the GitHub UI, since we have pushed directly in the main branch, a new even should have been triggered and the new action workflow should have started. To follow the action status, click on the _Actions_ tab again.

**9. Setting up Pages for the repository: almost there!**
If the above action workflow building were to be successful, then you should find a new branch under your repository called _gh-pages_, which it should contain the genereated structure of folders and html file for the Sphinx documentation. It should resemble the following:

![image](https://user-images.githubusercontent.com/26251075/137340487-73862787-fb8a-4ac2-81fb-59874dd74579.png)

If not, it means that something went wrong during the Sphinx action workflow. Check the _Actions_ tab. When a workflow is successful then a green check mark icon is shown, otherwise you would be presented with a red crossed icon:

![image](https://user-images.githubusercontent.com/26251075/137340888-9f888bf4-97d3-4ef0-bc99-47ec16a087fc.png)

Let's assume that everything went super smooth at your first attempt; it is then time to tell GitHub that you want to set up a web page for such repository. Here is the official [GitHub Pages tutorial](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site) for you to follow.

**Note:** Since we have pushed the html generated documentation on the _gh-pages_ branch, you need to select such path as settings for the repository associated GitHub page. So that's how it looks like for me:

![image](https://user-images.githubusercontent.com/26251075/137341786-ffe61797-f36b-42e8-9fe4-ee842157b094.png)

**10. That's it, well done!**

If you have made it so far, then congratulations! You have mange to create a repository that contains Sphinx based documentation, which it gets build and pushed automatically everytime a push or a pull-request event happens within the _main_ branch. As a result the generated html documenation gets pushed on a seperate branch, _gh-pages_, which in turns will be rendered as a web page by GitHub Pages.

![screen-gif](https://media1.tenor.com/images/ae2b9b09a0372639fab433a81fc1faae/tenor.gif?itemid=7620514)

**Note:** Unless there are specific errors not being addressed, it can take up to several minutes before GitHub servers refresh the page content. Untill then it keeps showing a "404 File not found" page

# Lesson learnt
If, like me, you wanted to have the generated html document structure being pushed on the same branch, that is _main_ (or _master_), then be prepare for a funky surprise which anyway can be reverted. In fact, the pre-built action completely removes the content of the current repository root directory and populates with all the Sphinx generated files and folder for the html build. If you navigate through the [commits history of my repo](https://github.com/carmat88/sphinx-to-gh-pages/commits/main) you can see that at a certain point the content of my `main` branch got overwritten at this [commit](https://github.com/carmat88/sphinx-to-gh-pages/tree/3697d6dd93fd59df9c622044724b6d2d5ddd5e9b).

Perhaps it may be possible to tweak the action code in order to create automatically a subfolder that would be pushed to main which contains the output of the Sphinx html build. Otherwise it seems however a standard practice to use a dedicated branch called `gh-pages` when it comes down to publishing html content for a repository.

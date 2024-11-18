# Mandatory Files

The mandatory files:

* `.documentary.toml` — the file used for repo connection to the Documentary Platform;
* `README.md` — a file that answers the "What", "Why" and "How" work in the repo;
* `CONTRIBUTING.md` — a file that explains how contribute to a repo and helps to verify that a well-formed pull request and opening useful issues are correctly submitted to the repo. This helps to understand how to contribute to the repo;
* `.md` files in `/docs` directory. The usable templates can be found in [`./docs-templates`](./docs-templates) directory;
* `CODEOWNERS` — a file that defines individuals or teams that are responsible for code in a repo. The code owners are automatically requested for review when someone opens a PR that modifies the code that they own. When someone with admin or owner permissions has enabled required reviews, they also require approval from a code owner before an author can merge the PR to the repo.


# Files Storage

* `.md` docs are stored in a root `docs/` directory;
* the files in format `.png`, `svg` and `.puml` are stored in an `image/` directory. When there are no images, use `/diagram` directory for `.puml`.

The structure for mandatory docs:

```markdown
(project root)/
├─ .github
│  ├─CODEOWNER 
├─ docs/
│  ├─ ayan.md
│  ├─ TOC.yaml
│  ├─ image/
│     ├─ processes.png
├─ .documentary.toml
├─ CONTRIBUTING.md
├─ README.md
```


## Files Naming

> **WARNING**
>
> Created file name is a part of a doc link or a doc section. These links can be shared with the teammates and posted in various spaces. So try to create a file name that unlikely will be changed in the future.

The requirements for files and directories naming:

* use lowercase;
* use hyphens, not underscores, to separate words, for example, `new-order.md`;
* use only standard ASCII alphanumeric characters in file and directory names;
* no spaces between the words of the file name;
* no capslock;
* no special symbols;
* try name the file same as the name of the section in the `TOC.yaml`.

Example:

```plaintext
- name: Home
      href: README.md
```


# PlantUML Service Diagrams

[PlantUML](https://plantuml.com) is used to create the service diagrams.

All system diagrams are stored and maintained in the `architecture-docs` repo in relevant service directories.

If you do not find any relevant diagrams for the service in the `architecture-docs` repo:

* consult the `README.md` in the `architecture-docs` repo;
* create a new service directory (if required);
* name the directory exactly like the name of the service;
* create your diagrams in this directory;
* assign a PR for the review by an architecture engineer;
* after the approval and merge, make the links of these diagrams to your docs.

> **NOTE**
>
> Make a snippet in a repo `README.md` file (how to make a snippet see in the [guideline](markdown-guideline.md) to add a  diagram

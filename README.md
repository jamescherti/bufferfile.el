# bufferfile.el - Rename, Delete, or Copy Files and Associated Buffers (e.g., clones/indirect buffers)
![Build Status](https://github.com/jamescherti/bufferfile.el/actions/workflows/ci.yml/badge.svg)
[![MELPA](https://melpa.org/packages/bufferfile-badge.svg)](https://melpa.org/#/bufferfile)
[![MELPA Stable](https://stable.melpa.org/packages/bufferfile-badge.svg)](https://stable.melpa.org/#/bufferfile)
![License](https://img.shields.io/github/license/jamescherti/bufferfile.el)
![](https://jamescherti.com/misc/made-for-gnu-emacs.svg)

This [bufferfile.el](https://github.com/jamescherti/bufferfile.el) package provides helper functions to delete, rename, or copy buffer files:
- `bufferfile-rename`: Renames the file visited by the current buffer, ensures that the destination directory exists, and updates the buffer name for all associated buffers, including clones/indirect buffers. It also ensures that buffer-local features referencing the file, such as Eglot or dired buffers, are correctly updated to reflect the new file name.
- `bufferfile-delete`: Delete the file associated with a buffer and kill all buffers visiting the file, including clones/indirect buffers.
- `bufferfile-copy`: Ensures that the destination directory exists and copies the file visited by the current buffer to a new file.

If this enhances your workflow, please show your support by **⭐ starring bufferfile.el on GitHub** to help more Emacs users discover its benefits.

**The *bufferfile* package overcomes limitations in Emacs' built-in functions:**

* **Emacs built-in renaming:** While indirect buffers continue to reference the correct file path, their buffer names can become outdated.
* **Emacs built-in deleting:** Indirect buffers are not automatically removed when the base buffer or another indirect buffer is deleted.

**The bufferfile package resolves these issues** by updating buffer names when a file is renamed and removing all related buffers, including indirect ones, when a file is deleted.

(To make *bufferfile* use version control when renaming or deleting files, you can set the variable `bufferfile-use-vc` to t. This ensures that file operations within *bufferfile* interact with the version control system, preserving history and tracking changes properly.)

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
## Table of Contents

- [bufferfile.el - Rename, Delete, or Copy Files and Associated Buffers (e.g., clones/indirect buffers)](#bufferfileel---rename-delete-or-copy-files-and-associated-buffers-eg-clonesindirect-buffers)
  - [Installation from MELPA](#installation-from-melpa)
  - [Usage](#usage)
  - [Customizations](#customizations)
    - [How to make Dired use bufferfile to rename files](#how-to-make-dired-use-bufferfile-to-rename-files)
    - [Making bufferfile use version control (VC), such as Git, when renaming or deleting files?](#making-bufferfile-use-version-control-vc-such-as-git-when-renaming-or-deleting-files)
    - [Hook functions](#hook-functions)
      - [Hooks for Renaming Files](#hooks-for-renaming-files)
      - [Hooks for Deleting Files](#hooks-for-deleting-files)
  - [Frequently asked questions](#frequently-asked-questions)
    - [What is the difference between bufferfile and the built-in Emacs rename and delete functions?](#what-is-the-difference-between-bufferfile-and-the-built-in-emacs-rename-and-delete-functions)
  - [Author and License](#author-and-license)
  - [Testimonials](#testimonials)
  - [Links](#links)

<!-- markdown-toc end -->


## Installation from MELPA

To install *bufferfile* from MELPA:

1. If you haven't already done so, [add MELPA repository to your Emacs configuration](https://melpa.org/#/getting-started).

2. Add the following code to your Emacs init file to install *bufferfile* from MELPA:
```emacs-lisp
(use-package bufferfile
  :ensure t
  :commands (bufferfile-copy
             bufferfile-rename
             bufferfile-delete)
  :custom
  ;; If non-nil, display messages during file renaming operations
  (bufferfile-verbose nil)

  ;; If non-nil, enable using version control (VC) when available
  (bufferfile-use-vc nil)

  ;; Specifies the action taken after deleting a file and killing its buffer.
  (bufferfile-delete-switch-to 'parent-directory))
```

## Usage

- To rename the current buffer's file and associated buffers, run: `M-x bufferfile-rename`
  (You will be prompted to enter a new name. The file will be renamed on disk, and the buffer—along with any associated buffers such as indirect buffers—will begin visiting the new file, with their buffer names updated accordingly.)

- To delete the current buffer's file and associated buffers, run: `M-x bufferfile-delete`
  (You will be asked to confirm the deletion. If confirmed, the file will be removed from disk, and all associated buffers, including indirect buffers, will be killed.)

- To copy the current buffer's file, run: `M-x bufferfile-copy`

## Customizations

### How to make Dired use bufferfile to rename files

By default, Dired's rename operation (`R`) updates the file on disk but may not correctly update any buffers visiting the file, especially for renamed files with indirect (clone) buffers.

To address this, you can override Dired's rename keybinding (`R`) to use `bufferfile-dired-do-rename`, which uses bufferfile rename functions. This ensures that all associated buffers, including indirect ones, are properly updated after the rename operation:

```elisp
;; Override Dired's rename behavior to use bufferfile rename functions,
;; ensuring buffers visiting the renamed file are updated accordingly.
(with-eval-after-load 'dired
  (define-key dired-mode-map (kbd "R") #'bufferfile-dired-do-rename))
```

### Making bufferfile use version control (VC), such as Git, when renaming or deleting files?

To make *bufferfile* use version control (VC) when renaming or deleting files, you can set the variable `bufferfile-use-vc` to `t`. This ensures that file operations within *bufferfile* interact with the version control system, preserving history and tracking changes properly.

``` emacs-lisp
(setq bufferfile-use-vc t)
```

### Hook functions

The *bufferfile* package provides customizable hook variables that allow users to execute functions before and after renaming or deleting files. These hooks can be used to integrate additional logic, such as logging, or updating dependent buffers.

#### Hooks for Renaming Files

- **`bufferfile-pre-rename-functions`**
  A list of functions executed before renaming a file.
  Each function receives three arguments:
  - `previous-path`: The original file path.
  - `new-path`: The new file path.
  - `list-buffers`: The list of buffers associated with the file.

- **`bufferfile-post-rename-functions`**
  A list of functions executed after a file has been renamed.
  Each function receives the same three arguments as `bufferfile-pre-rename-functions`.

#### Hooks for Deleting Files

- **`bufferfile-pre-delete-functions`**
  A list of functions executed before a file is deleted.
  Each function receives two arguments:
  - `path`: The file path to be deleted.
  - `list-buffers`: The list of buffers associated with the file.

- **`bufferfile-post-delete-functions`**
  A list of functions executed after a file has been deleted.
  Each function receives the same two arguments as `bufferfile-pre-delete-functions`.

## Frequently asked questions

### What is the difference between bufferfile and the built-in Emacs rename and delete functions?

Here are the limitations of Emacs' built-in functions:

- Renaming: Indirect buffers point to the correct file path, but their buffer names become outdated.
- Deleting: Indirect buffers are not removed when the base buffer or another indirect buffer is deleted.

**The bufferfile package addresses these issues** by ensuring that buffer names are updated when renaming a file and that all buffers, including indirect buffers, are deleted when a file is removed.

These limitations of the built-in functions motivated the development of *bufferfile*, which improves support for renaming and deleting buffers, including indirect buffers.

## Author and License

The *bufferfile* Emacs package has been written by [James Cherti](https://www.jamescherti.com/) and is distributed under terms of the GNU General Public License version 3, or, at your choice, any later version.

Copyright (C) 2024-2026 James Cherti

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version. This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details. You should have received a copy of the GNU General Public License along with this program.

## Testimonials

- [tinkerorb](https://www.reddit.com/r/emacs/comments/1jngj4b/comment/mlii6fw/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button): I can't even remember for how long I have wanted something that solves this (but I also have not lifted a single finger of my own to solve it for myself). Thank you for this package!

## Links

- [bufferfile.el @GitHub](https://github.com/jamescherti/bufferfile.el)

Other Emacs packages by the same author:
- [minimal-emacs.d](https://github.com/jamescherti/minimal-emacs.d): This repository hosts a minimal Emacs configuration designed to serve as a foundation for your vanilla Emacs setup and provide a solid base for an enhanced Emacs experience.
- [compile-angel.el](https://github.com/jamescherti/compile-angel.el): **Speed up Emacs!** This package guarantees that all .el files are both byte-compiled and native-compiled, which significantly speeds up Emacs.
- [outline-indent.el](https://github.com/jamescherti/outline-indent.el): An Emacs package that provides a minor mode that enables code folding and outlining based on indentation levels for various indentation-based text files, such as YAML, Python, and other indented text files.
- [vim-tab-bar.el](https://github.com/jamescherti/vim-tab-bar.el): Make the Emacs tab-bar Look Like Vim’s Tab Bar.
- [easysession.el](https://github.com/jamescherti/easysession.el): Easysession is lightweight Emacs session manager that can persist and restore file editing buffers, indirect buffers/clones, Dired buffers, the tab-bar, and the Emacs frames (with or without the Emacs frames size, width, and height).
- [elispcomp](https://github.com/jamescherti/elispcomp): A command line tool that allows compiling Elisp code directly from the terminal or from a shell script. It facilitates the generation of optimized .elc (byte-compiled) and .eln (native-compiled) files.
- [tomorrow-night-deepblue-theme.el](https://github.com/jamescherti/tomorrow-night-deepblue-theme.el): The Tomorrow Night Deepblue Emacs theme is a beautiful deep blue variant of the Tomorrow Night theme, which is renowned for its elegant color palette that is pleasing to the eyes. It features a deep blue background color that creates a calming atmosphere. The theme is also a great choice for those who miss the blue themes that were trendy a few years ago.
- [Ultyas](https://github.com/jamescherti/ultyas/): A command-line tool designed to simplify the process of converting code snippets from UltiSnips to YASnippet format.
- [dir-config.el](https://github.com/jamescherti/dir-config.el): Automatically find and evaluate .dir-config.el Elisp files to configure directory-specific settings.
- [flymake-bashate.el](https://github.com/jamescherti/flymake-bashate.el): A package that provides a Flymake backend for the bashate Bash script style checker.
- [flymake-ansible-lint.el](https://github.com/jamescherti/flymake-ansible-lint.el): An Emacs package that offers a Flymake backend for ansible-lint.
- [inhibit-mouse.el](https://github.com/jamescherti/inhibit-mouse.el): A package that disables mouse input in Emacs, offering a simpler and faster alternative to the disable-mouse package.
- [quick-sdcv.el](https://github.com/jamescherti/quick-sdcv.el): This package enables Emacs to function as an offline dictionary by using the sdcv command-line tool directly within Emacs.
- [enhanced-evil-paredit.el](https://github.com/jamescherti/enhanced-evil-paredit.el): An Emacs package that prevents parenthesis imbalance when using *evil-mode* with *paredit*. It intercepts *evil-mode* commands such as delete, change, and paste, blocking their execution if they would break the parenthetical structure.
- [stripspace.el](https://github.com/jamescherti/stripspace.el): Ensure Emacs Automatically removes trailing whitespace before saving a buffer, with an option to preserve the cursor column.
- [persist-text-scale.el](https://github.com/jamescherti/persist-text-scale.el): Ensure that all adjustments made with text-scale-increase and text-scale-decrease are persisted and restored across sessions.
- [pathaction.el](https://github.com/jamescherti/pathaction.el): Execute the pathaction command-line tool from Emacs. The pathaction command-line tool enables the execution of specific commands on targeted files or directories. Its key advantage lies in its flexibility, allowing users to handle various types of files simply by passing the file or directory as an argument to the pathaction tool. The tool uses a .pathaction.yaml rule-set file to determine which command to execute. Additionally, Jinja2 templating can be employed in the rule-set file to further customize the commands.
- [kirigami.el](https://github.com/jamescherti/kirigami.el): The *kirigami* Emacs package offers a unified interface for opening and closing folds across a diverse set of major and minor modes in Emacs, including `outline-mode`, `outline-minor-mode`, `outline-indent-minor-mode`, `org-mode`, `markdown-mode`, `vdiff-mode`, `vdiff-3way-mode`, `hs-minor-mode`, `hide-ifdef-mode`, `origami-mode`, `yafolding-mode`, `folding-mode`, and `treesit-fold-mode`. With Kirigami, folding key bindings only need to be configured **once**. After that, the same keys work consistently across all supported major and minor modes, providing a unified and predictable folding experience.

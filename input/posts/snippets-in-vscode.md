Title: VSCode Snippets 
Published: 18/7/2024 11:00
Tags: [markdown, vscode] 
---

I use Visual Studio Code to write markdown notes of my meetings. I datetime every meeting to be able to find it afterwards. One of the shorthands I use is 'dt'. I always forget how to set it up when I reinstall my vscode, so here's the instructions from github copilot chat, which explained how to set it up again!

So now I know I can ask Copilot how to do it, but I still want to document it, so here it is: 

### Github Copilot Instructions

To recreate the shorthand command in Visual Studio Code for inserting the current date and time in a Markdown file using `ctrl-space`, you can use a snippet. Here's how to set it up:

1. Open the Command Palette with `Ctrl+Shift+P`.
2. Type `Configure User Snippets` and select it.
3. Choose `markdown.json` for Markdown files.
4. Insert the following snippet configuration:

```json
"Insert Current Date and Time": {
    "prefix": "dt",
    "body": [
        "### $CURRENT_YEAR.$CURRENT_MONTH.$CURRENT_DATE $CURRENT_HOUR:$CURRENT_MINUTE:$CURRENT_SECOND"
    ],
    "description": "Inserts the current date and time"
}
```

5. Save the `markdown.json` file.

Now, when you type `dt` in a Markdown file and press `Ctrl+Space`, it should suggest your snippet. Selecting it will insert the current date and time formatted as you described.

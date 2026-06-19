## Heads Up
Hey there, before we start this article I would like to inform you that this is not a technological blog post. It's mostly about the tools I use and that I like to configure every now and then. So in case you are looking for a technological blog post, "Bye".



## Terminal: Your Gateway to the Command Line
I use Debian as my daily driver for the last two years and before that I was using i3 tiling window manager, so all in all I use Linux for my work.

And that is the reason I like to configure my terminal to a point where I could do pretty much everything with it. Now this doesn't mean I use neovim or curl or wget all the time.

So every time I setup a new distro in my virtual machine, I run a series of installation scripts to make my terminal look more or less like it was in my previous machines.

**Common Terminal Emulators:**
- GNOME Terminal: Default on most GNOME-based distributions
- Konsole: KDE's feature-rich terminal emulator
- Alacritty: GPU-accelerated, blazingly fast
- Kitty: Modern terminal with image and font support
- Terminator: Grid-based terminal with splits
- xterm: Lightweight, classic choice

## Zsh: The Shell with Superpowers
I don't think anybody who uses Linux doesn't know about zsh. Maybe you don't use it but it's very hard to not know about it.

In short, it is a shell different from bash that provides you a ton of customization. Don't think it's superior to bash. It's like the engine of a car—its sole job is to process commands. What makes it different is the customization that it offers.

**What Zsh Brings to the Table:**
- Advanced globbing and pattern matching
- Improved history with search and recall
- Associative arrays and advanced scripting features
- Better completion system with context-aware suggestions
- Theming and prompt customization options 


## Oh-My-Zsh: Making Customization Effortless
Now customization is one thing, but what if you make the customizing part so hard that everyone just uses it as it is? Which basically means you are using zsh but it works like bash. No point in using zsh when it's exactly like bash. Oh-My-Zsh is a framework developed for zsh so it becomes easy for users to customize it. It offers a plugin system so anyone could create a plugin for Oh-My-Zsh, and with just one command you can install it, enable it, and run it. It's that simple.

**Installing and Using Plugins:**
```bash
# Install a plugin by adding it to ~/.zshrc
plugins=(git node npm oh-my-zsh-nvm)

# Load zsh to activate plugins
exec zsh
```

**Popular Plugins to Get Started:**
- git: Shortcuts and git aliases for common commands
- node/npm: Node.js and npm completions
- docker: Docker command completions
- extract: Quick extraction for various archive formats

**My Oh-My-Zsh Setup:**

I keep my plugin list lean and focused on tools I actually use. Here's what I have enabled in my `.zshrc`:

```bash
export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME="robbyrussell"

plugins=(
  git
  npm
  node
  docker
  zsh-autosuggestions
  zsh-syntax-highlighting
)

source $ZSH/oh-my-zsh.sh
```

The theme is minimal, and I let Starship handle the fancy prompt customization instead. This keeps things fast and clean.

## Starship: Craft Your Perfect Prompt
In this AI world we do prompting a lot. But in the terminal, we call prompt something else. The information you see before your cursor blinks is called the prompt. Generally, if you don't configure much of your terminal, you get to see which directory you are in. But I like to know a few more things, so I use Starship. It's not the only tool that customizes the prompt, but it's good and fast too.

**Example Starship Prompt:**
```bash
anuj@debian ~/Repos/discoveries/discoveries  master  (main)  
❯
```

**What Starship Can Show:**
- Current git branch and commit status
- Programming language versions (Node.js, Python, Rust, etc.)
- Execution time of the last command
- Battery status
- Time and date
- Custom symbols and colors based on context

**My Starship Configuration:**

I use a Gruvbox dark color palette with a multi-line format that shows my OS, username, directory, git info, programming languages, and time. Here's a snippet of my `.config/starship.toml`:

```toml
"$schema" = 'https://starship.rs/config-schema.json'

format = """
[](color_orange)\
$os\
$username\
[](bg:color_yellow fg:color_orange)\
$directory\
[](fg:color_yellow bg:color_aqua)\
$git_branch\
$git_status\
[](fg:color_aqua bg:color_blue)\
$nodejs\
$python\
$rust\
[](fg:color_blue bg:color_bg3)\
$docker_context\
[](fg:color_bg3 bg:color_bg1)\
$time\
[ ](fg:color_bg1)\
$line_break$character"""

palette = 'gruvbox_dark'

[palettes.gruvbox_dark]
color_fg0 = '#fbf1c7'
color_bg1 = '#3c3836'
color_bg3 = '#665c54'
color_blue = '#458588'
color_aqua = '#689d6a'
color_orange = '#d65d0e'
color_yellow = '#d79921'
```

**Starship Tip:**
The key to a good prompt is balance. I show only the most relevant information (git branch, language versions, time) without cluttering. You can customize each module's format individually to match your workflow.

Here's a [screenshot of my starship setup](https://github.com/JackCompass/googleme/blob/master/images/screenfetch.png) in action.

